## Android annotations注解原理

注解的定义是能够添加到java源代码的语法数据。类、方法、变量、参数都可以被注解，在android中常常可以看到的有@Override,@Deprecated 前者表示重写父类方法，后者表示注解元素过时了。<br>
注解的原理是：定义注解，然后在AbstractProcessor中通过处理该注解（运行时注解需要通过java反射处理注解），利用JavaWriter生成我们所需要的java代码。<br>

本文档结合github例子：https://github.com/MerlinYu/PreferenceAnnotation 说明。

#### 定义注解

	//PreferenceField.class
	@Retention(RetentionPolicy.SOURCE)
	@Target(ElementType.FIELD)
	public @interface PreferenceField {
	  String defaultValue() default "";
	}
	
	@Retention(RetentionPolicy.SOURCE)
	@Target(ElementType.TYPE)
	public @interface PreferenceItem {
	  String tableName() default "preference_annotation";
	}


###### @Retention<br>
这个注解表示注解也就是@PreferenceField保留的方式：

	public enum RetentionPolicy {
	     SOURCE,
    	 CLASS,
    	 RUNTIME
    }
   
SOURCE:只保留在源码中，不保留在class中，同时也不加载到虚拟机中。<br>
CLASS:保留在源码中，同时也保留到class中，但是不加载到虚拟机中。 <br>
RUNING:保留到源码中，同时也保留到class中，最后加载到虚拟机中。<br>
在编译期时可以SOURCE和CLASS的注解，而在运行期时处理RUNING注解。

###### @Target(ElementType.FIELD) <br>
这个注解表示注解的元素的类型，其值可以有TTPE(类),FIELD(变量)，METHOD(方法),PARAMETER(参数)...<br>

在PreferenceField.class中定义了一个PreferenceField类变量注解，这个变量只保留在源码中，其默认值为""。


### java编译期处理注解

java提供了一个类可以用来处理java注解：AbstractProcessor，我们只需要继承这个类然后实现Abstract方法即可。
文档中对AbstractProcessor的说明：

>AbstractProcessor
>>
 * An abstract annotation processor designed to be a convenient
 * superclass for most concrete annotation processors.  This class
 * examines annotation values to compute the {@linkplain
 * getSupportedOptions options}, {@linkplain
 * getSupportedAnnotationTypes annotation types}, and {@linkplain
 * getSupportedSourceVersion source version} supported by its
 * subtypes.


##### 定义AbstractProcessor

其源码：https://github.com/MerlinYu/PreferenceAnnotation/blob/master/preference-processor/src/main/java/com/lucky/PreferenceProcessor.java

在前面的代码中我们定义了两个注解：类注解：PreferenceItem，变量注解：PreferenceField，这两个注解一块使用，其目的是自动处理android SharedPreference.

	//AccountPreference.java
	@PreferenceItem( tableName = "account")
	public class AccountPreference {
	  @PreferenceField
	  public String name;
	
	  @PreferenceField
	  public String account;
	
	  @PreferenceField( defaultValue = "true")
	  public boolean ok;
	
	  @PreferenceField( defaultValue = "10")
	  public int kitty;
	
	}
	
###### AbstractProcessor
定义PreferenceProcessor继承AbstractProcessor处理注解。


	@AutoService(Processor.class)
	public class PreferenceProcessor extends AbstractProcessor {
	
	}

@AutoService 在编译期时表示自动运行PreferenceProcessor。如何使用网上有些博客可以参考，这时不再详述，本篇博客只是阐明一个实现的思路。<br>
在process中处理注解：

	  
	  @Override
	  public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
	    if (roundEnv.processingOver()) {
	      return true;
	    }

	    Set<? extends Element> preferenceFields = roundEnv.getElementsAnnotatedWith(PreferenceField.class);
	    Map<Element, List<Element>> preferenceObjects = new HashMap<>(20);
	    //select PreferenceItem and PreferenceField list put them in map
	    for (final Element field : preferenceFields) {
	      final Element classElement = field.getEnclosingElement();
	      if (classElement.getKind() != ElementKind.CLASS) {
	        sendErrorMessage(classElement, ERROR_MESSAGE, PreferenceItem.class.getSimpleName());
	        return true;
	      }
	      if (classElement.getAnnotation(PreferenceItem.class) != null) {
	        List<Element> list = preferenceObjects.get(classElement);
	        if (list == null) {
	          list = new ArrayList<>();
	          preferenceObjects.put(classElement,list);
	        }
	        list.add(field);
	      }
	    }
	    .....
    }

这段代码的含义是拿到PreferenceItem对应的PreferenceField，将两都放在一个map当中，然后再对Map当中的元素做出处理，具体的可以参考源代码。
编译过后生成的代码将在：<br>
![](https://github.com/MerlinYu/blog/blob/master/blog_file/android/android_annotations.png)


















