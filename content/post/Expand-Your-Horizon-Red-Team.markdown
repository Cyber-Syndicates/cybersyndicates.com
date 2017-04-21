+++
author = "Alexander Rymdeko-Harvey"
tags = []
title = "Expand Your Horizon Red Team – Modern SaaS C2"
description = "Python WSGI C2"
date = "2017-04-21T21:39:41-05:00"
+++

Well to say that the past month or so for C2 infrastructure has been impactful, would be an understatement. Domain fronting has added yet another card to the deck of tricks the Red Team can deploy. Before this all took place, I was putting in some time to broaden the horizon of how the Red Team could deploy easy, yet effective C2. During this research, I found an interesting solution to this problem. Deploy our C2 redirectors on platforms that often are not intended to have console access. Hopefully to open the horizon of providers and IP space to operate / host from. 

Using Python Flask applications and providers that support WSGI, I was able to put together three new projects to demonstrate this TTP.

*TL;DR* Source code and projects are at the end of the post. The following video shows from nothing to fully functional redirector in one minute flat:

<iframe src="https://player.vimeo.com/video/213955360?autoplay=1&loop=1" width="640" height="368" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/213955360">One Minute C2 Redirector</a> from <a href="https://vimeo.com/user52367620">Alexander Rymdeko-Harvey</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

For reference we will be covering the following flask redirectors:

* https://github.com/killswitch-GUI/flask_heroku_redirector
* https://github.com/killswitch-GUI/flask_pythonanywhere_redirector
* https://github.com/killswitch-GUI/flask_appengine_redirector

## Native Expansion
Extensive work has been done by @bluscreenofjeff to filter HTTP traffic via apache mod_rewrite, working off this research I knew Python would also be able to handle this. Using a Python based platform the possibilities are nearly endless in conjunction with providers like Heroku. Sporting the CLI interface, Heroku commands can be used to update system environment variables remotely. This allows for extremely agile manipulation of the redirectors, without the need of multiple stages to filter requests. 

## Providers 
limited VPS provides has been an area of concern for myself in the past. Many of the big-name core VPS providers have gotten a bad reputation when it comes to IP blacklisting. Also, one single point of failure to discover all your C2 to pivot off could take place. Adding a few more options to the mix is a great way to throw off defenders, or help spread out your offensive end points. Plus, who is going guess you’re running you C2 from PythonAnywhere.

## Deployment
Time is often limited and spending more time than necessary on C2 often means more time off target. A really cool benefit of using SaaS C2 is the ease of code deployments; often we see conventional use cases ranging from quickly standing up a file server to expanding existing infrastructure. This concept makes deploying code a cinch on many platforms, Heroku is a great example of this. If you walk through the setup process of the flask_heroku_redirector, you will notice simple environmental variables to push the configs (Ex. heroku config:set FORWARD_DOMAIN=cybersyndicates.com). This proves to be very powerful to quickly block, change or even pull down payloads you don’t want the Blue Team to hit. 

## IP Space 
A simple WHOIS query can say a ton about a domain, it will expose potential owner, address and corporation. We can setup WHOIS guard, but that is only half the picture; we need to also look at the IP space we originate from as well. An analyst determining if the traffic is malicious will be asking (Who owns this? Why is this client talking to it? What type of resource?). When we start talking about our operational footprint, considerations like where I host from is a big deal. Take for example third party C2, its near undetectable and will easily circumvent most analytics. The return on investment to blend in and pass the sniff test can be invaluable to sustaining a foot hold. The flask_appengine_redirector project is still dependent on other factors such as a domain name to match the ploy, but provides this initial top cover from operating in Google IP space, without the extent of third party C2. 

## Technical Overview 
While each project has its own specifics from the platform dependent configuration files, the core code base is nearly identical across multiple platforms. The first thing you will notice is the common use of OS environment variables. This is common practice for encryption, secret and private keys on these platforms. We use them to allow us to quickly change and alter our C2  redirector, either through the web interface or provider CLI.

```
# STATIC VAR
__DEBUG = bool(os.environ.get('DEBUG', True))
__FORWARD_DOMAIN = str(os.environ.get('FORWARD_DOMAIN', 'cybersyndicates.com'))
__CURRENT_DOMAIN = str(os.environ.get('CURRENT_DOMAIN', 'test-1234555555.herokuapp.com'))
```

Flask is a great example and standard configuration looks as follows: 

set port to forward on (Defaults to 80):

* `heroku config:set PORT=80` 

set debug output to the log outlet (Defaults to True):

* `heroku config:set DEBUG=True` 

set forward C2 domain (Where to redirect to):

* `heroku config:set FORWARD_DOMAIN=cybersyndicates.com`

set redirector domain (Incoming domain name):

* `heroku config:set CURRENT_DOMAIN=tranquil-dawn-50102.herokuapp.com`

Once you move on to the core code components you will notice that I also added in a “debug” class to feed your request to. This is very helpful to see the data being received by your C2 since you will not have tools like tcpdump available. 

```
class debug_request():
    """
    debug printing of request object.
    """
    def __init__(self):
        # setup DB connection
        # TODO: add db support
        # self.conn = sql
        pass

    def log_request(self, request_obj):
        """
        used to log full request to db or other logging outlet
        """
        # TODO: build full logic for request
        req = self.build_json(request_obj)
        pass
    
    def print_request(self, request_obj):
        """
        print the entire request in json pprint
        """
        print self.build_json(request_obj)

    def build_json(self, request_obj):
        """
        build the json obj
        """
        json_dict = {}
        json_dict['url'] = request_obj.url
        json_dict['method'] = request_obj.method
        json_dict['headers'] = {key: value for (key, value) in request.headers if key != 'Host'}
        json_dict['method'] = request_obj.cookies
        json_dict['data'] = request_obj.data
        return json.dumps(json_dict, ensure_ascii=False, indent=4, sort_keys=True)
```

you should expect to see some verbose logging until you set your debug to false; here is an example of that logging console:

```
2017-04-21T22:05:50.027323+00:00 app[web.1]: proxy response: <Response 14718 bytes [200 OK]>
2017-04-21T22:05:50.098293+00:00 app[web.1]: {
2017-04-21T22:05:50.098295+00:00 app[web.1]:     "data": "",
2017-04-21T22:05:50.098296+00:00 app[web.1]:     "headers": {
2017-04-21T22:05:50.098297+00:00 app[web.1]:         "Accept": "image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5",
2017-04-21T22:05:50.098298+00:00 app[web.1]:         "Accept-Encoding": "gzip, deflate",
2017-04-21T22:05:50.098299+00:00 app[web.1]:         "Accept-Language": "en-us",
2017-04-21T22:05:50.098299+00:00 app[web.1]:         "Connect-Time": "0",
2017-04-21T22:05:50.098300+00:00 app[web.1]:         "Connection": "close",
2017-04-21T22:05:50.098301+00:00 app[web.1]:         "Cookie": "_ga=GA1.3.1533808371.1492657513; __atuvc=1%7C16",
2017-04-21T22:05:50.098301+00:00 app[web.1]:         "Referer": "https://vast-meadow-71019.herokuapp.com/",
2017-04-21T22:05:50.098302+00:00 app[web.1]:         "Total-Route-Time": "0",
2017-04-21T22:05:50.098303+00:00 app[web.1]:         "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/603.1.30 (KHTML, like Gecko) Version/10.1 Safari/603.1.30",
2017-04-21T22:05:50.098303+00:00 app[web.1]:         "Via": "1.1 vegur",
2017-04-21T22:05:50.098304+00:00 app[web.1]:         "X-Forwarded-For": "108.44.160.205",
2017-04-21T22:05:50.098305+00:00 app[web.1]:         "X-Forwarded-Port": "443",
2017-04-21T22:05:50.098305+00:00 app[web.1]:         "X-Forwarded-Proto": "https",
2017-04-21T22:05:50.098306+00:00 app[web.1]:         "X-Request-Id": "514fade7-8222-494f-92f0-388100a01a0f",
2017-04-21T22:05:50.098307+00:00 app[web.1]:         "X-Request-Start": "1492812350097"
2017-04-21T22:05:50.098307+00:00 app[web.1]:     },
2017-04-21T22:05:50.098308+00:00 app[web.1]:     "method": {
2017-04-21T22:05:50.098308+00:00 app[web.1]:         "__atuvc": "1%7C16",
2017-04-21T22:05:50.098309+00:00 app[web.1]:         "_ga": "GA1.3.1533808371.1492657513"
2017-04-21T22:05:50.098310+00:00 app[web.1]:     },
2017-04-21T22:05:50.098310+00:00 app[web.1]:     "url": "https://vast-meadow-71019.herokuapp.com/logo.png"
2017-04-21T22:05:50.098311+00:00 app[web.1]: }
2017-04-21T22:05:50.098316+00:00 app[web.1]: https://cybersyndicates.com/logo.png
``` 

One of the powerful aspects of Flask is the ability to use “Custom Decorators” with your Flask app routes. We will be intercepting a “request” object before it hits a flask route. Using a custom decorator we are able to achieve this:

```
@app.before_request
@proxy_required
def log_request():
    """
    log request and force proxy
    """
    try:
        log_outlet = {'address' : request.remote_addr, 'url' : request.base_url}
        print log_outlet
    except:
        return make_response(jsonify({'error': 'somthing went wrong (contact admin)'}), 400)

@app.route('/<string:request_path>', methods=['GET'])
def request_forward(request_path):
    return "works/n"
```

If you look at the above code we have a custom decorator declared as “@app.before_request”, this tells flask to run this code route before sending to the standard route views. Using a custom decorator we “wrap” this function with “@proxy_required”, to intercept the request and manipulate the data:

```
def proxy_required(func):
    """
    proxy http requests to forward domain (our real C2).
    """
    @wraps(func)
    def wrapper(*args, **kwargs):
        """
        wrapper function with decorator
        """
        try:
            if __DEBUG:
                debug_req.print_request(request)
                print request.url.replace(__CURRENT_DOMAIN, __FORWARD_DOMAIN)
            # setup our proxy for C2
            resp = requests.request(
                method=request.method,
                url=request.url.replace(__CURRENT_DOMAIN, __FORWARD_DOMAIN),
                headers={key: value for (key, value) in request.headers if key != 'Host'},
                data=request.get_data(),
                cookies=request.cookies,
                allow_redirects=True)
            excluded_headers = ['content-encoding', 'content-length', 'transfer-encoding', 'connection']
            headers = [(name, value) for (name, value) in resp.raw.headers.items()
                       if name.lower() not in excluded_headers]
            response = Response(resp.content, resp.status_code, headers)
            print "proxy response: %s" % (response)
            return response
            if not response:
                # FORWARD RESPONSE TO SERVER
                return func(*args, **kwargs)
        except:
            return make_response(jsonify({'error': 'somthing went wrong (contact admin)'}), 400)
    return wrapper
```
The code will replace the requested URL with the forwarded domain or IP, fix up the request object by pulling the header values and data within the request to build a new request object. Using this object we send it to designated server with all the data and proper headers. This acts as a proxy and returns this entire response object to the original requester. 

Here is a great video of it in action with CobaltStrike: 

<iframe src="https://player.vimeo.com/video/214249797?autoplay=1&loop=1" width="640" height="368" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/214249797">Flask Proxy C2 Redirector</a> from <a href="https://vimeo.com/user52367620">Alexander Rymdeko-Harvey</a> on <a href="https://vimeo.com">Vimeo</a>.</p>


Flask PythonAnywhere C2 redirector template:
<div class="github-card" data-github="killswitch-gui/flask_pythonanywhere_redirector" data-width="400" data-height="153" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

Google App Engine Flask C2 redirector:
<div class="github-card" data-github="killswitch-gui/flask_appengine_redirector" data-width="400" data-height="153" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

flask heroku C2 redirector template:
<div class="github-card" data-github="killswitch-gui/flask_heroku_redirector" data-width="400" data-height="153" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>
