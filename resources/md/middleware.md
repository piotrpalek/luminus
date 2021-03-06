## Adding custom middleware

Since Luminus uses Ring for routing the application handler
is a standard Ring handler and can be wrapped in middleware
just like you would in any other Ring based application.

The middleware allows wrapping the handlers in functions which can modify the the way the request is processed. 
Middleware functions are often used to extend the base functionality of Ring handlers to match the needs of 
the particular application.

A middleware is simply a function which accepts an existing handler with some optional parameters and returns a new handler with some added behaviour. An example of a middleware function would be:

```clojure
(defn wrap-nocache [handler]
  (fn [request]
     (let [response (handler request)]
        (assoc-in response [:headers  "Pragma"] "no-cache"))))
```

As you can see the wrapper accepts the handler and returns a function which in turn accepts the request. Since the returned function was defined in the scope where the handler exists, it can use it internally. When called, it will call the handler with the request and add Pragma: no-cache to the response map. For detailed information please refer to the official [Ring documentation](https://github.com/ring-clojure/ring/wiki).


The middleware is added in the `middleware` namespace of your project. Any development middleware, such as middleware for showing stacktraces, should be added in the `development-middleware` function. It will only be invoked when the `:dev` envrionment key is set. Only add middleware that you also wish to use in production in the `production-middleware` function.

```clojure
(defn development-middleware [handler]
  (if (env :dev)
    (-> handler
        wrap-error-page
        wrap-exceptions)
    handler))

(defn production-middleware [handler]
  (-> handler
      (wrap-restful-format :formats [:json-kw :edn :transit-json :transit-msgpack])
      (wrap-idle-session-timeout
        {:timeout (* 60 30)
         :timeout-response (redirect "/")})
      (wrap-defaults
        (assoc-in site-defaults [:session :store] (memory-store session/mem)))
      wrap-servlet-context
      wrap-internal-error))
```    

## Useful ring middleware

* [ring-ratelimit](https://github.com/myfreeweb/ring-ratelimit) - Rate limiting middleware
* [ring-etag-middleware](https://github.com/mikejs/ring-etag-middleware) - Calculates etags for ring responses and returns 304 responses when appropriate
* [ring-gzip-middleware](https://github.com/mikejs/ring-gzip-middleware) - Gzips ring responses for user agents which can handle it
* [ring-upload-progress](https://github.com/joodie/ring-upload-progress) - Provide upload progress data in ring session
