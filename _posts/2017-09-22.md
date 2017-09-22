### 关于ZipException: Not in GZIP format的问题

<p>
前几天在开发对接相关接口的时候需要实现一个Base64解码和gzip解压缩的功能，Base64自然用到的apache的common包，还要实现一个gzip压缩和解压缩的工具类，
因为是用java开发，就需要用到GZIPInputStream和GZIPOutputStream这两个类，写完之后大概代码如下:
</p>
<pre><code>
public static String compress(String str) throws IOException { 
       if (str == null || str.length() == 0) { 
           return str; 
       } 
       ByteArrayOutputStream out = new ByteArrayOutputStream(); 
       GZIPOutputStream gzip = new GZIPOutputStream(out); 
       gzip.write(str.getBytes()); 
       gzip.close();
       return out.toString("UTF-8"); 
}
public static String decompress(String str) throws IOException { 
         if (str == null || str.length() == 0) { 
            return str; 
         }
         GZIPInputStream gis = new GZIPInputStream(new ByteArrayInputStream(str.getBytes("UTF-8"))); 
         BufferedReader bf = new BufferedReader(new InputStreamReader(gis, "UTF-8")); 
         String outStr = ""; 
         String line; 
         while ((line=bf.readLine())!=null) { 
               outStr += line; 
         } 
         return outStr; 
  }
</code></pre>
<p>测试时发现的错误ZipException:</p>
<pre>
<code>
java.util.zip.ZipException: Not in GZIP format
	at java.util.zip.GZIPInputStream.readHeader(GZIPInputStream.java:165)
	at java.util.zip.GZIPInputStream.<init>(GZIPInputStream.java:79)
	at java.util.zip.GZIPInputStream.<init>(GZIPInputStream.java:91)
</code>
<code>
// Check header magic
        if (readUShort(in) != GZIP_MAGIC) {
            throw new ZipException("Not in GZIP format");
        }
</code>
</pre>
<p>
然后重新review了一下代码，并没有找出错误，而GZIPInputStream中对应错误地方的代码(JDK8中164行)：
byte数组怎么会出错呢，百思不得其解，无奈只能求助于google，在StackOverflow上找到一句话，大概的意思是说：
工程师们经常会混淆String和byte[]，应在合适的地方选择正确的类型；只有在相同的os和encoding时才能正确转换，
而且也不是所有的bytes都能转换为String；而且在字符串经过base64和gzip的过程中，多了一步将byte[]和String的转换过程造成了错误，
将这一步造成了byte[]的不正确，修改代码后正确运行。</p>
<pre><code>
 public static byte[] compress(String str) throws IOException { 
        ...... 
 }
 public static String decompress(byte[] bytes) throws IOException { 
        ......
 }
 </code></pre>
<p>
参考网址：
[1]: https://stackoverflow.com/questions/14466840/java-error-creating-a-gzipinputstream-not-in-gzip-format
[2]: http://www.cnblogs.com/searcherY/p/6723615.html
</p>



