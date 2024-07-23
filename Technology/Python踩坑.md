# Python踩坑

## 一. python的函数传参

### 1、kwargs的参数接收问题

- ```python
    def func(param, *args, **kwargs):
    	pass
    ```

- **kwargs能且只能接收到如下形式的传参

    ```python
    func(param, query="what is ChatGPT")
    ```

    此时

    ```python
    kwargs: {"query": "what is ChatGPT"}
    ```

    

## 二、线程和进程

### 1、flask中使用scrapy框架

- 在flask框架下尝试使用CrawlerProcess启动scrapy框架的爬虫，但是收到报错signal only works in main thread of the main interpreter

- 为了解决这个问题，您可以将 `CrawlerProcess` 运行在一个单独的进程中，而不是 Flask 应用程序的线程中。这可以通过使用 Python 的 `multiprocessing` 模块实现。

    下面是一个简单的示例代码，可以帮助您在 Flask 应用程序中使用 `CrawlerProcess`:

    ```python
    from flask import Flask
    from scrapy.crawler import CrawlerProcess
    from multiprocessing import Process
    
    app = Flask(__name__)
    
    def run_crawler():
        process = CrawlerProcess()
        process.crawl(MySpider)
        process.start()
    
    @app.route('/')
    def index():
        p = Process(target=run_crawler)
        p.start()
        p.join()
        return 'Crawler started'
    
    if __name__ == '__main__':
        app.run()
    ```

- 如果run_crawler有返回值如何获取

- 如果您希望从 `run_crawler` 函数中返回结果并在 Flask 应用程序中使用它，您可以使用 Python 的 `Queue` 模块来实现进程间通信。以下是一个示例代码：

    ```python
    from flask import Flask
    from scrapy.crawler import CrawlerProcess
    from multiprocessing import Process, Queue
    
    app = Flask(__name__)
    
    def run_crawler(queue):
        process = CrawlerProcess()
        process.crawl(MySpider)
        process.start()
        queue.put('Crawler finished')
    
    @app.route('/')
    def index():
        q = Queue()
        p = Process(target=run_crawler, args=(q,))
        p.start()
        result = q.get()
        p.join()
        return result
    
    if __name__ == '__main__':
        app.run()
    ```

- 在这个示例中，`run_crawler` 函数接受一个 `Queue` 对象作为参数，并在运行结束时将结果放入队列中。在 Flask 应用程序的路由函数中，我们创建一个 `Queue` 对象并将其作为参数传递给 `run_crawler` 函数。然后我们启动进程并等待它完成。当进程完成时，我们从队列中获取结果并将其作为响应返回给客户端。

    请注意，在此示例中，我们仅仅将一个简单的字符串作为结果放入队列中，如果你的返回值是一个复杂对象或者需要在多个进程中共享数据，那么您可能需要更复杂的方式来进行进程间通信。