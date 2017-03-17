#####来聊一聊如何搭建一个支付框架<br>
在搭建支付框架时首先要考虑：可能拥有多种支付方式，不同的支付方式支付参数不一样，全局只允许有一个支付实例，支付结果如何返回？<br>
基于以上的考虑我将采用单例模式（全局只有一个支付类）、工厂模式（实现多种支付方式）和观察者模（支付结果）式来实现整个支付框架。<br>

#####支付类 
######SupportPayment 支付名称类

    public class SupportPayment implements Parcelable{
     /**支付唯一标识*/
     private int actionID;
     /**支付名称*/
     private String name;

    private SupportPayment(int actionID, String name) {
      this.actionID = actionID;
      this.name = name;
     }
    }
  

######PaymentAction 支付接口
	public interface PaymentAction {
	  void pay(String orderID, Map<String, String> parameters);
	}

接下来要做的事应该是为某个支付类SupportPayment生成相对应的支付接口PaymentAction<br>
我在SupportPayment中定义一个静态方法来生成相应的支付接口，实际上采用简单工厂的模式的思想，根据SupportPayment来生成对应的支付接口。<br>

	    public static PaymentAction toPaymentAction(Activity activity, 
                                              SupportPayment actionType) {
	    switch (actionType.actionID) {
	      case ID_ALIPAY:
	        return new AliPaymentAction(activity);
	      case ID_WECHAT:
	        return new WeChatPaymentAction(activity);
	    }
	    return null;
  	}
AliPaymentAction 和 WeChatPaymentAction是实现了PaymentAction支付接口的支付实例。其相关的代码如下：<br>
AliPaymentAction<br>

	public class AliPaymentAction implements PaymentAction{
	
	  private Activity activity;
	  public AliPaymentAction(Activity atx) {
	    activity = atx;
	  }
	
	
	  @Override
	  public void pay(String orderID, Map<String, String> parameters) {
	  	//    if cancel
		//    PaymentManager.getInstance(activity).payCancel();
		//    if success
		//    PaymentManager.getInstance(activity).paySuccess();
		//    if failed
		//    PaymentManager.getInstance(activity).payFailed();
	
	  }
	}

WeChatPaymentAction<br>


	public class WeChatPaymentAction implements PaymentAction {
	
	  private Activity activity;
	
	  public WeChatPaymentAction(Activity activity) {
	    this.activity = activity;
	  }
	
	
	  @Override
	  public void pay(String orderID, Map<String, String> parameters) {
	  
	  	//    if cancel
		//    PaymentManager.getInstance(activity).payCancel();
		//    if success
		//    PaymentManager.getInstance(activity).paySuccess();
		//    if failed
		//    PaymentManager.getInstance(activity).payFailed();
	
	  }
	}

在这里我们并不关心具体是怎么支付的只是说明如何找找寻一个支付的框架。<br>

######PaymentManager 支付Manager 
	public class PaymentManager {

	  private static volatile PaymentManager instance;
	  private Context context;	
	
	  private PaymentManager(Context context) {
	    this.context = context.getApplicationContext();
	  }
	
	  public static PaymentManager getInstance(Activity activity) {
	    if (instance == null) {
	      synchronized (PaymentManager.class) {
	        if (instance == null) {
	          instance = new PaymentManager(activity);
	        }
	      }
	    }
	    return instance;
	  }
	}
	
在上面的代码中用单例双重锁的形式实现了PaymentManager的单例，在app内只有一个支付的Manager.<br>

######PaymentResultCallback 支付结果

	public interface PaymentResultCallback {
    void onPaymentSuccess(SupportPayment actionType, String orderId,
                            Map<String, String> parameters);

    void onPaymentFailed(int errCode, String errMessage, SupportPayment actionType, String orderId,
                          Map<String, String> parameters);

    void onPaymentCancel(SupportPayment actionType, String orderId,
                          Map<String, String> parameters);
  	}
  	
定义支付结果Result接口，在接口中我们定义了三种方法，success, cancel, failed,固定的参数是订单orderId, parameters中添加支付需要的额外的参数。<br>
PaymentManager中需要调用支付，并将结果返回，将PaymentManger修改如下：<br>

	/**
	 * Created by yuchao on 2/7/17.
	 */
	
	public class PaymentManager {
	
	  private static volatile PaymentManager instance;
	  private Context context;
	  private Handler mainHandler;
	
	  private HashMap<String, WeakReference<PaymentResultCallback>> payCallbacks;
	
	  private PaymentManager(Context context) {
	    this.context = context.getApplicationContext();
	    mainHandler = new Handler(context.getMainLooper());
	    payCallbacks = new HashMap<>();
	  }
	// 单例
	  public static PaymentManager getInstance(Activity activity) {
	    if (instance == null) {
	      synchronized (PaymentManager.class) {
	        if (instance == null) {
	          instance = new PaymentManager(activity);
	        }
	      }
	    }
	    return instance;
	  }
	// map key
	  public String buildPayBackKey(SupportPayment actionType, String orderID) {
	    StringBuilder builder = new StringBuilder(actionType.getPayName());
	    builder.append("?order=" + orderID);
	    return builder.toString();
	  }
	
	
	//  支付
	  public void pay(@NonNull Activity activity, @NonNull String orderID, @NonNull SupportPayment actionType,
	                  Map<String, String> parameters, PaymentResultCallback callback) {
	    if (callback == null) {
	      throw new IllegalArgumentException("payment action callback can't be null");
	    }
	    PaymentAction paymentAction = SupportPayment.toPaymentAction(activity, actionType);
	    if (paymentAction != null) {
	//      PaymentResultCallback 加入到Map列表当中,为支付结果回调做准备
	      payCallbacks.put(buildPayBackKey(actionType, orderID), new WeakReference<>(callback));
	      paymentAction.pay(orderID, parameters);
	    }
	  }
	
	// 支付成功
	  public void paySuccess(SupportPayment actionType, String orderID,
	                         Map<String, String> parameters) {
	    String key = buildPayBackKey(actionType, orderID);
	    WeakReference<PaymentResultCallback> callbackRef = payCallbacks.remove(key);
	    if (callbackRef != null) {
	      final PaymentResultCallback callback = callbackRef.get();
	      if (callback != null) {
	        mainHandler.post(() -> {
	          callback.onPaymentSuccess(actionType, orderID, parameters);
	        });
	      }
	    }
	  }
	  // 支付失败
	  public void payFailed(int errorCode, String errMsg, SupportPayment actionType, String orderID,
	                        Map<String, String> parameters) {
	    String key = buildPayBackKey(actionType, orderID);
	    WeakReference<PaymentResultCallback> callbackRef = payCallbacks.remove(key);
	    if (callbackRef != null) {
	      final PaymentResultCallback callback = callbackRef.get();
	      if (callback != null) {
	        mainHandler.post(() -> {
	          callback.onPaymentFailed(errorCode, errMsg, actionType, orderID, parameters);
	          ;
	        });
	      }
	    }
	  }
	//取消支付
	  public void payCancel(SupportPayment actionType, String orderID,
	                        Map<String, String> parameters) {
	    String key = buildPayBackKey(actionType, orderID);
	    WeakReference<PaymentResultCallback> callbackRef = payCallbacks.remove(key);
	    if (callbackRef != null) {
	      final PaymentResultCallback callback = callbackRef.get();
	      if (callback != null) {
	        mainHandler.post(() -> {
	          callback.onPaymentCancel(actionType, orderID, parameters);
	          ;
	        });
	      }
	    }
	  }
	}
	
上面的代码不用做详细解释就可以看懂。
之前的代码中定义了PaymentManager, PaymentResultCallback, SupportPayment,PaymentAction。再继续要做的事情就是去调用它，去进行支付。<br>
调用实例：<br>
	
	public void payOrder() {
	    String orderId = "55034";
	    SupportPayment payment = SupportPayment.AliPay;
	    Map<String, String> parameters = new HashMap<>();
	    parameters.put("time", String.valueOf(System.currentTimeMillis()));
	    PaymentManager.PaymentResultCallback callback = new PaymentManager.PaymentResultCallback() {
	      @Override
	      public void onPaymentSuccess(SupportPayment actionType, String orderId, Map<String, String> parameters) {
	        //TODO: success
	      }

	      @Override
	      public void onPaymentFailed(int errCode, String errMessage, SupportPayment actionType, String orderId, Map<String, String> parameters) {
	        //TODO: failed
	      }
	
	      @Override
	      public void onPaymentCancel(SupportPayment actionType, String orderId, Map<String, String> parameters) {
	        //TODO: cancel
	      }
	    };

	    PaymentManager.getInstance(this).pay(this, orderId, payment, parameters, callback);

	  }
	  
一个简单的支付框架就是这样的，如果有新的支付方式加进来只需要实现接口PaymentAction，并在SupportPayment toPaymentAction方法中添加相应的case代码，就可以了。GitHub地址：<https://github.com/MerlinYu/structure/tree/master/app/src/main/java/com/structure/pay>

	
