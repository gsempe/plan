---
date: 2019-07-26
tags: golang
---

Function [http.FileServer](https://godoc.org/net/http#FileServer) is very handy to serve content from a file system. One weakness of feature of the http package is that if a request does not match an existing file `http.FileServer` serves a builtin, non customizable 404 HTTP error. cf. [toHTTPError(err error) (msg string, httpStatus int)](https://github.com/golang/go/blob/465d821ecfa567167e72420c17fb78c8aabb4bf9/src/net/http/fs.go#L628-L637) private function in http pkg.

To overcome this weakness it's possible to wrap


### 1) [Response to: Custom 404 with Gorilla Mux and std http.FileServer](https://stackoverflow.com/a/26142515/21052)


The FileServer is generating the 404 response. The FileServer handles all requests passed to it by the mux including requests for missing files. There are a few ways to to serve static files with a custom 404 page:

- Write  your own file handler using [ServeContent](http://godoc.org/net/http#ServeContent). This handler can generate error responses in whatever way you want. It's not a lot of code if you don't generate index pages.
- Wrap the [FileServer](http://godoc.org/net/http#FileServer) handler with another handler that hooks the [ResponseWriter](http://godoc.org/net/http#ResponseWriter) passed to the FileHandler. The hook writes a different body when [WriteHeader(404)](http://godoc.org/net/http#ResponseWriter.WriteHeader) is called.
- Register each static resource with the mux so that not found errors are handled by the catchall in the mux. This approach requires a simple wrapper around [ServeFile](http://godoc.org/net/http#ServeFile).

Here's a sketch of the wrapper described in the second approach:

```go
type hookedResponseWriter struct {
    http.ResponseWriter
    ignore bool
}

func (hrw *hookedResponseWriter) WriteHeader(status int) {
    hrw.ResponseWriter.WriteHeader(status)
    if status == 404 {
	    hrw.ignore = true
	    // Write custom error here to hrw.ResponseWriter
    }
}

func (hrw *hookedResponseWriter) Write(p []byte) (int, error) {
    if hrw.ignore {
	    return len(p), nil
    }
    return hrw.ResponseWriter.Write(p)
}

type NotFoundHook struct {
    h http.Handler
}

func (nfh NotFoundHook) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    nfh.h.ServeHTTP(&hookedResponseWriter{ResponseWriter: w}, r)
}
```

Use the hook by wrapping the FileServer:

```go
 router.PathPrefix("/public/").Handler(NotFoundHook{http.StripPrefix("/public/", http.FileServer(http.Dir("public")))})
```

One caveat of this simple hook is that it blocks an optimization in the server for copying from a file to a socket.

### 2) [Response to: How to custom handle a file not being found when using go static file server?](https://stackoverflow.com/a/47286697/21052)

The handler returned by [`http.FileServer()`][1] does not support customization, it does not support providing a custom 404 page or action.

What we may do is wrap the handler returned by `http.FileServer()`, and in our handler we may do whatever we want of course. In our wrapper handler we will call the file server handler, and if that would send a `404` not found response, we won't send it to the client but replace it with a redirect response.

To achieve that, in our wrapper we create a wrapper [`http.ResponseWriter`][2] which we will pass to the handler returned by `http.FileServer()`, and in this wrapper response writer we may inspect the status code, and if it's `404`, we may act to _not_ send the response to the client, but instead send a redirect to `/index.html`.

This is an example how this wrapper `http.ResponseWriter` may look like:

    type NotFoundRedirectRespWr struct {
    	http.ResponseWriter // We embed http.ResponseWriter
    	status              int
    }
    
    func (w *NotFoundRedirectRespWr) WriteHeader(status int) {
    	w.status = status // Store the status for our own use
    	if status != http.StatusNotFound {
    		w.ResponseWriter.WriteHeader(status)
    	}
    }
    
    func (w *NotFoundRedirectRespWr) Write(p []byte) (int, error) {
    	if w.status != http.StatusNotFound {
    		return w.ResponseWriter.Write(p)
    	}
    	return len(p), nil // Lie that we successfully written it
    }

And wrapping the handler returned by `http.FileServer()` may look like this:

    func wrapHandler(h http.Handler) http.HandlerFunc {
    	return func(w http.ResponseWriter, r *http.Request) {
    		nfrw := &NotFoundRedirectRespWr{ResponseWriter: w}
    		h.ServeHTTP(nfrw, r)
    		if nfrw.status == 404 {
    			log.Printf("Redirecting %s to index.html.", r.RequestURI)
    			http.Redirect(w, r, "/index.html", http.StatusFound)
    		}
    	}
    }

Note that I used `http.StatusFound` redirect status code instead of `http.StatusMovedPermanently` as the latter may be cached by browsers, and so if a file with that name is created later, the browser would not request it but display `index.html` immediately.

And now put this in use, the `main()` function:

    func main() {
    	fs := wrapHandler(http.FileServer(http.Dir(".")))
    	http.HandleFunc("/", fs)
    	panic(http.ListenAndServe(":8080", nil))
    }

Attempting to query a non-existing file, we'll see this in the log:

    2017/11/14 14:10:21 Redirecting /a.txt3 to /index.html.
    2017/11/14 14:10:21 Redirecting /favicon.ico to /index.html.

Note that our custom handler (being well-behaviour) also redirected the request to `/favico.ico` to `index.html` because I do not have a `favico.ico` file in my file system. You may want to add this as an exception if you don't have it either.

The full example is available on the [Go Playground][3]. You can't run it there, save it to your local Go workspace and run it locally.

Also check this related question: https://stackoverflow.com/questions/34017342/log-404-on-http-fileserver

  [1]: https://golang.org/pkg/net/http/#FileServer
  [2]: https://golang.org/pkg/net/http/#ResponseWriter
  [3]: https://play.golang.org/p/5E9KreSsnp