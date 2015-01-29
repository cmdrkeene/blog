---
layout:     post
title:      "Buffer Your Golang HTML Templates, Or Else"
subtitle:   "Safer usage of the standard html/template package"
date:       2015-01-23 12:00:00
author:     "Brandon Keene"
header-img: "img/post-bg-02.jpg"
---

So I ran into a sharp edge today inside Go's
[html/template](http://golang.org/pkg/html/template/) package.

If you learn about [Writing Web Applications](https://golang.org/doc/articles/wiki/) 
from the standard documentation, you might not see the problem right away. That 
document creates a helper func to render HTML templates, and handle any errors 
that come occur:

{% highlight go %}
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, err := template.ParseFiles(tmpl + ".html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    err = t.Execute(w, p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
  }
}
{% endhighlight %}

You might expect (incorrectly as we'll see) that the client will receive 500 on 
__any__ `Execute()` error. But not always! Because `Execute()` only cares about
an `io.Writer` it is totally fine returning an error at any point in its
`Write()` process.

This [Play example](http://play.golang.org/p/fBcfegyZEB) shows two
`template.Execute()` failures that result in a different HTTP status code. This
snippet shows you the basic idea:

{% highlight go %}
{% raw %}

// broken template "foo"
src := `{{.Foo}}`

// an http.ResponseWriter
w := httptest.NewRecorder()

// returns 200 on error
t := template.Must(template.New("foo").Parse(src))
err := t.Execute(w, nil)
if err != nil {
  w.WriteHeader(500) // doesn't happen
}

// returns 500 on error
t = template.Must(template.New("foo").Parse(src))
err = t.Execute(w, []struct{}{}) // note the empty struct
if err != nil {
  w.WriteHeader(500) // acutally happens
}
{% endraw %}
{% endhighlight %}

This happens because we're writing an HTTP response using an
`http.ResponseWriter`. Even though we haven't processed the whole template,
we've written some bytes and therefore already sent a 200 OK. 

From the docs:

{% highlight go %}
type ResponseWriter interface {
    // ...
    // If WriteHeader has not yet been called, Write calls 
    // WriteHeader(http.StatusOK) before writing the data.
    // ...
    Write([]byte) (int, error)
}
{% endhighlight %}

It's not ideal to call the request a success if the template failed to execute.
To fix this, you can buffer your template results before you respond:

{% highlight go %}
var b bytes.Buffer
err = template.Execute(&b, p)
if err != nil {
  http.Error(w, err.Error(), http.StatusInternalServerError)
  return
}
b.WriteTo(w) // 200 OK
{% endhighlight %}

The documentation for `(*template.Template) Execute()` mentions this:

{% highlight go %}
// If an error occurs executing the template or writing its output,
// execution stops, but partial results may already have been written to the 
// output writer.
{% endhighlight %}

This is sadly something that you "just must know". A close reading of __all__
the docs is required to know how `http.ResponseWriter` actually works. It would
be nice if [Writing Web Applications](https://golang.org/doc/articles/wiki/) at
least mentioned this in the somewhat misleading example above.

Tags: #golang, #go, #programming, #bugs, #webapps