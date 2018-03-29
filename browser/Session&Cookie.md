## 前言
```Session```和```Cookie```作为web开发中两个重要的"工具"，对其进行一次全面的了解是非常有必要的。（以下所说的内容都是基于web开发条件下的，Cookie本身是一个概念，Web Cookie只是其中的一种实现）

## why Cookie, why Session

- why Cookie

  为什么我们要使用Cookie呢？众所周知，http是一种无状态的协议，也就是说，每次的HTTP请求对于服务器来说都是一个全新的请求，如果我们想携带一些客户之前的信息，如果不做任何操作是不可能实现的。

  Cookie就是用来解决这个问题的，由服务端通过HTTP响应头Set-Cookie字段设置，客户端来存储，每次请求的时候依存于HTTP请求头部的Cookie字段，发送给后端当做一种状态。

  - 举个栗子，你开了一家炸鸡店，一天来了一个长得很好看的小姐姐，跟你说她要一份A套餐。那么我们可以发现，为了优化客户体验，每次只要小姐姐一进店，我们就可以直接给他一份A套餐，免去她点餐说话的时间。这时候我们送给小姐姐一个精致的蝴蝶发卡，让她戴在头上，以后无论谁在点餐，只要看到发卡，就是给他一份A套餐，那么这个发卡就是Cookie，记录了小姐姐的爱好是A套餐。

- why Session

  那么为什么还有Session呢？Session其实是Cookie的一种具体实现。每次客户登录网站的时候，后台生成一个Session并设置一个过期时间，并用一个hash存储，在响应的HTTP请求里设置一个```会话阶段的Cookie```。下次后台如果接收的HTTP请求里的Cookie在hash里有对应，那么我们就认为这是一个已经登录的用户，直接为他提供服务。

  - why 会话阶段Cookie

    Cookie是有过期时长的，如果不设置的话那么就认为是会话阶段的Cookie，是存储在浏览器进程中的，如果我们关闭了浏览器，存储在浏览器进程中的Cookie自然已经被删除了，下一次发送HTTP请求的时候自然就不会带有Cookie，服务器发现头部没有Cookie，就会在服务器端重新生成一个Session并存储在hash中，而之前的那个对应的Session则会在过期时间到了之后自动删除。如果想做记住登录这种操作，那么需要设置一个过期时间，就会存储在客户端的硬盘中了。

  - 再举个栗子，还是这个小姐姐，但是她并不愿意接受我们的发卡。作为一个万能的程序员，我们只能用一下高科技手段了，每当这个小姐姐进屋，我们就在她头顶全息投影一个发卡，无论点餐员是谁，也都可以直接给她一份A套餐，她离开的时候就取消掉投影。这个仅在店里有效的就好比一个Session


##  知识点梳理

  - Cookie是存储在客户端的，如果不设置过期时长，则是存储在浏览器进程中的，彻底退出浏览器后就该Cookie就会消失;设置了过期时长，是存储在客户端硬盘中的，在过期时间之前有效。

  - Session只是会话期Cookie的一种实现方式，在服务器端会存储一份hash列表保存，在浏览器端作为Cookie保存。

  - 一般来说Session都是Http Only的Cookie，用来简单防范XSS攻击，因为js使用document.cookie无法获取Http Only的Cookie

  - 当Cookie的过期时间被设定时，设定的日期和时间只与客户端相关，而不是服务端

  - Cookie 可以设置key=value，可以设置domain,path,expires,Secure,HttpOnly 具体可见MDN

  - http的站点无法使用Secure的Cookie

##  js操作Cookie

  - github 8.9k的库 [js-cookie](https://github.com/js-cookie/js-cookie)

  - 简单封装
    ```javascript
    function cookie () {
      function read (name) {
        var match = document.cookie.match(new RegExp('(^|;\\s*)(' + name + ')=([^;]*)'));
        return (match ? decodeURIComponent(match[3]) : null);
      }

      function write ({ name, value, expires, path, domain, secure }) {
        var cookie = [];
        cookie.push(name + '=' + encodeURIComponent(value));

        if (typeof expires === 'number') {
          cookie.push('expires=' + new Date(expires).toGMTString());
        }

        if (typeof path === 'string') {
          cookie.push('path=' + path);
        }

        if (typeof domain === 'string') {
          cookie.push('domain=' + domain);
        }

        if (secure === true) {
          cookie.push('secure');
        }

        document.cookie = cookie.join('; ');
      }

      function remove (name) {
        write({
          name: name,
          value: '',
          expires: Date.now() - 86400000
        });
      }

      return {
        read: read,
        write: write,
        remove: remove
      }
    }

    ```

##  参考
  - [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
