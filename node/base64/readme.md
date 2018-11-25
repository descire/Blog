# base64

  Base66是一种用64个字符来表示任意二进制数据的方法。

  当我们打开二进制文件时，看到一大堆乱码，因为二进制文件中包含很多无法显示和打印的字符，这时就需要二进制编码方法 -- Base64。

  对于二进制数据的处理是每3个字节一组，也就是一组24bit，将其划分为4组，也就是每组6bit，正好表示64个字符。

  Base64通过将3字节的二进制数据编码为4字节的文本数据，长度会增加33%，好处是直接显示。

  如果二进制不满足3字节，那么就需要用\x00字节在末尾补足后，再在编码的末尾加上1个或者2个=号表示补了多少个字节，解码时会自动去掉。