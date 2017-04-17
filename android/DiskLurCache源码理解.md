### DiskLurCache源码理解

DiskLurCache是被google认可的硬盘缓存方案。最近看了一下其源码，发现其实现很巧妙。<br>
基本思路是这样的：通过一个LinkedHashMap记录缓存的Key和缓存文件路径Entry数据。通过journal文件记录对缓存数据的操作，包括创建（DIRTY）移除(REMOVE)发布(CLEAN)，其格式是这样的：

	  	 *     libcore.io.DiskLruCache
	     *     1
	     *     100
	     *     2
	     *
	     *     CLEAN 3400330d1dfc7f3f7f4b8d4d803dfcf6 832 21054
	     *     DIRTY 335c4c6028171cfddfbaae1a9c313c52
	     *     CLEAN 335c4c6028171cfddfbaae1a9c313c52 3934 2342
	     *     REMOVE 335c4c6028171cfddfbaae1a9c313c52
   
当新建一个DiskLurCache会读取journal文件，将已经缓存的数据key和Entry添加到LinkedHashMap当中，当添加缓存时LinkedHashMap.put(key, entry), 当读取缓存时通过linkedHashMap.get(key)来得到缓存文件的Entry的相关信息。

#### DiskLurCache的构造方法：<br>

    mDiskLruCache = DiskLruCache.open(cacheDir, getAppVersion(context), 1, CACHE_SIZE);
  
	 /**
	   * Opens the cache in {@code directory}, creating a cache if none exists
	   * there.
	   *
	   * @param directory a writable directory
	   * @param valueCount the number of values per cache entry. Must be positive.
	   * @param maxSize the maximum number of bytes this cache should use to store
	   * @throws IOException if reading or writing the cache directory fails
	   */
	  public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
	      throws IOException {
	 			 ...... 
	      	    cache.readJournal();
    		    cache.processJournal();
    
	      }
代码中的注释已经很明确的说明构造函数参数的作用。当appVersion发生改变时会将缓存清空。在构造函数中有一个参数：valueCount，这个参数的含义是针对同一个key可以存储几条数据，我们一般使valueCount = 1。
readJournal() 读取缓存记录，而processJournal()计算缓存大小，并清理垃圾数据。

#### 缓存数据

	   DiskLruCache.Editor editor = mDiskLruCache.edit(hashKey);
	   OutputStream outputStream = editor.newOutputStream(0);
	   outputStream.write(value.getBytes());
	   outputStream.flush();
	   outputStream.close();
	   editor.commit();

从DiskLurCache中获取Editor然后获取OutputStream再向其写入byte数据。其内部代码是：
	
	private synchronized Editor edit(String key, long expectedSequenceNumber) throws IOException {
	
    checkNotClosed();
    validateKey(key);
    Entry entry = lruEntries.get(key);
    ....

    Editor editor = new Editor(entry);
    entry.currentEditor = editor;
    
    // Flush the journal before creating files to prevent file leaks.
    journalWriter.write(DIRTY + ' ' + key + '\n');
    journalWriter.flush();
    return editor;
    }
    
  
#### 读取数据
	
	 DiskLruCache.Snapshot snapShot = mDiskLruCache.get(hashKey);
	 
  	 InputStream is = snapShot.getInputStream(0);
        StringBuilder out = new StringBuilder();
        mByteBuffer = new byte[CACHE_BYTE_SIZE];
        int n;
        while ((n = is.read(mByteBuffer)) != -1) {
          out.append(new String(mByteBuffer, 0, n));
        }
        mByteBuffer = null;
        return out.toString();
        
 读取数据时从DiskLurCache中获取Snapshot然后从中获取InoutStream再从中读取byte数据。
 其内部源码是：
 
	  /**
	   * Returns a snapshot of the entry named {@code key}, or null if it doesn't
	   * exist is not currently readable. If a value is returned, it is moved to
	   * the head of the LRU queue.
	   */
	  public synchronized Snapshot get(String key) throws IOException {
	    Entry entry = lruEntries.get(key);
	    InputStream[] ins = new InputStream[valueCount];
		....
      for (int i = 0; i < valueCount; i++) {
        ins[i] = new FileInputStream(entry.getCleanFile(i));
      }
    return new Snapshot(key, entry.sequenceNumber, ins, entry.lengths);
    }
    
 在上面的存取缓存的代码中获取InputStream和OutputStream中都需要输入一个index,
 
	 editor.newOutputStream(0)
	 snapShot.getInputStream(0)

这个index 与valueCount悉悉相关，我们知道valueCount的含义是同一个key可以存储几条数据，也就是同一个key有几个InputStream和OutputStream。
DiskLurCache在内部通过ThreadPoolExecutor去清理缓存：

	 if (journalRebuildRequired()) {
	      executorService.submit(cleanupCallable);
	  }
	    

	/** This cache uses a single background thread to evict entries. */
	  final ThreadPoolExecutor executorService =
	      new ThreadPoolExecutor(0, 1, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
	  private final Callable<Void> cleanupCallable = new Callable<Void>() {
	    public Void call() throws Exception {
	      synchronized (DiskLruCache.this) {
	        if (journalWriter == null) {
	          return null; // Closed.
	        }
	        trimToSize();
	        if (journalRebuildRequired()) {
	          rebuildJournal();
	          redundantOpCount = 0;
	        }
	      }
	      return null;
	    }
	  };
	  
就这是DiskLurCache的关键源码。其他的都比较简单了。




