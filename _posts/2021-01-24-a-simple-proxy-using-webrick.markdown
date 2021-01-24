---
layout: post
title: a simple web proxy using WEBrick
date: 2021-01-24
tags:
  - ruby
  - webrick
---

As the [README](https://github.com/ruby/webrick) states, "WEBrick is an HTTP server toolkit that can
be configured as an HTTPS server, a proxy server, and a virtual-host server". I recently needed to
create a small proxy, with minimal effort, in order to hide some functionality of a proof-of-concept
API. WEBrick is quite powerful, and a small proxy does not require more than 20-25 lines or code.

The [AbstractServlet](https://docs.ruby-lang.org/en/2.7.0/WEBrick/HTTPServlet/AbstractServlet.html)
class allows us to respond to `GET`, `HEAD` and `OPTIONS` requests. We can use it to encapsulate our
logic of receiving a request, forwarding it to the server (possibly changing it) and responding back.

Let's write a simple proxy, that receives a request, appends a custom query param and forwards it to
the API server `https://api-server.com`. We will append the `realm` query parameter with a hardcoded
value of `qa-realm`.

First, let's create our proxy as a subclass of `AbstractServlet`. We will implement the 
[do_GET](https://docs.ruby-lang.org/en/2.7.0/RDoc/Servlet.html#method-i-do_GET) method as we care only
for `GET` requests. The incoming request object is stored in the `request` parameter, we will manipulate
this later in order to add our query parameter. In order to respond, we need to set the `content_type` and
body values on the `response` object.

```ruby
require "webrick"

class MyProxy < WEBrick::HTTPServlet::AbstractServlet
  def do_GET(request, response)
    response.content_type = "text/plain"
    response.body = "It works!"
  end
end
```

Next, we will add the realm parameter. We need to parse the request URL in order to make sure that we
handle correctly all use cases involving any existing parameters. We will use [URI](https://rubyapi.org/2.7/o/uri)
for this.

```ruby
require "webrick"
require "uri"

class MyProxy < WEBrick::HTTPServlet::AbstractServlet
  REALM = "qa-realm"

  def do_GET(request, response)
    uri = forwarded_uri(request.unparsed_uri)

    response.content_type = "text/plain"
    response.body = "It works! New URI is #{uri}"
  end

  private

  def forwarded_uri(unparsed_uri)
    uri = URI(unparsed_uri)
    params = URI.decode_www_form(uri.query || "") << ["realm", REALM]
    uri.query = URI.encode_www_form(params)
    uri.to_s
  end
end
```

Right now, we manipulate the request URI, but we don't forward anything to the intended endpoint.
We will use [Net::HTTP](https://rubyapi.org/2.7/o/net/http) to forward the request and pass the response
back. We will also use the body, and the content type we retrieve from the proxied server when we pass
the response back.

```ruby
require "webrick"
require "net/http"
require "uri"

class MyProxy < WEBrick::HTTPServlet::AbstractServlet
  HOST = "api-server.com"
  REALM = "qa-realm"

  def do_GET(request, response)
    uri = forwarded_uri(request.unparsed_uri)

    http = Net::HTTP.new(HOST, 443)
    http.use_ssl = true
    resp = http.request(Net::HTTP::Get.new(uri))
    body = resp.body

    response.content_type = resp["content-type"]
    response.body = body
  end

  private

  def forwarded_uri(unparsed_uri)
    uri = URI(unparsed_uri)
    params = URI.decode_www_form(uri.query || "") << ["realm", REALM]
    uri.query = URI.encode_www_form(params)
    uri.to_s
  end
end
```

In order to run our proxy, we need a few more missing pieces. First we need to create a new
[WEBrick::HTTPServer](https://docs.ruby-lang.org/en/2.7.0/WEBrick.html#module-WEBrick-label-Starting+an+HTTP+server)

```ruby
server = WEBrick::HTTPServer.new(:Port => ENV["PORT"] || 8080)
```

Then we need to mount our proxy under an endpoint, we can use the root endpoint or any other we want.

```ruby
server.mount "/", MyProxy
```

Finally, let's allow stopping the server using Ctrl+C and then start the server.

```ruby
trap("INT"){ server.shutdown }
server.start
```

If we run our proxy with `ruby myproxy.rb` it will start serving using port 8080 under /.

```text
[2021-01-24 21:36:00] INFO  WEBrick 1.6.0
[2021-01-24 21:36:00] INFO  ruby 2.7.1 (2020-03-31) [x86_64-darwin18]
[2021-01-24 21:36:00] INFO  WEBrick::HTTPServer#start: pid=47104 port=8080
```

The final code is the following:

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require "webrick"
require "net/http"
require "uri"

class MyProxy < WEBrick::HTTPServlet::AbstractServlet
  HOST = "api-server.com"
  REALM = "qa-realm"

  def do_GET(request, response)
    uri = forwarded_uri(request.unparsed_uri)

    http = Net::HTTP.new(HOST, 443)
    http.use_ssl = true

    resp = http.request(Net::HTTP::Get.new(uri))
    body = resp.body

    response.content_type = resp["content-type"]
    response.body = body
  end

  private

  def forwarded_uri(unparsed_uri)
    uri = URI(unparsed_uri)
    params = URI.decode_www_form(uri.query || "") << ["realm", REALM]
    uri.query = URI.encode_www_form(params)
    uri.to_s
  end
end

server = WEBrick::HTTPServer.new(:Port => ENV["PORT"] || 8080)
server.mount "/", MyProxy

trap("INT"){ server.shutdown }
server.start
```
