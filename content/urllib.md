Title: Urllib2 可能比 Urllib 阻塞得更久
Date: 2012-02-11 13:19
Category: blog
Summary:

socket._fileobject 的读缓冲区默认大小(_rbufsize)是 8192，也就是说调用 readline() 时，默认情况下它会先读 8KB 的数据到 buffer，然后再从中取出一行数据返回给调用方。如果关闭了读缓冲，readline() 会一个字节接一个字节的读数据，只要读到 \n 就立即返回。

假设一个 HTTP 响应头中的 Transfer-Encoding 是 chunked，并且每块 chunk 都是一行小于 8KB 的数据，然后客户端希望一收到 chunk 就立即对其进行处理。但是因为 readline() 希望先读 8KB 的数据，只要不够 8KB 就一直阻塞，直到收到足够的 chunk 为止(或者 EOF)。

而 urllib.urlopen 和 urllib2.urlopen 创建 socket._fileobject 的行为是不一样的，urllib 关闭了读缓冲，而 urllib2 则使用了默认的 8KB 大小的缓冲区。

所以第一次调用 urllib2.urlopen(URL).readline() 时，会比第一次调用 urllib.urlopen(URL).readline() 慢得多。如果服务器针对第二块 chunk 的处理需要较长时间的话，那 urllib2 的 readline() 就会一直阻塞在第一次的 readline() 调用上，哪怕已经收到了一行数据也是如此。要解决这个问题只要在 readline() 前把 response.fp._rbufsize 设为 0 就可以了。