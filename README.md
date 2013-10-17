facebook xmpp webclient
=======================

Example code to go with my blog post on how to connect to facebook chat with your own webclient


## Facebook Chat via Punjab Bosh and XMPP
I recently setup a web based facebook chat and found a number of problems with the information provided on the web on how to do this.  This post serves as a reminder to myself and hopes to help others in my situation<br>
### Here is a quick diagram on the technology stack
![stack](http://1.bp.blogspot.com/-9H-uiq5jRSA/Ul7DHOIXFII/AAAAAAAAE9A/7n88WaVW3dE/s1600/xmpp_stack.png "stack")

## Steps to making a web based client talk to through facebook chat
First thing to understand is that you will not be able to talk to facebooks (somewhat XMPP compliant) server directly through your web browser.  You are going to have to go through and intermediary service known as a BOSH server.  Some more information on [BOSH here](http://en.wikipedia.org/wiki/BOSH).

**Step 1:** Setup and configure your BOSH service.  In doing a bit of research I found that most people are using <a href="https://github.com/twonds/punjab">punjab</a>.  Punjab is written in python and runs on top of [Twisted](http://twistedmatrix.com/trac/), here are the steps to getting your environmant setup.
On the server you want to run the BOSH service:<br>

* Download Python 2.7 [here](http://www.python.org/getit/). (I have heard of problems with the above stack in 3.3)
* Next (and yes this is required) download and install pyOpenSSL for python 2.7.  Download link [here](https://pypi.python.org/pypi/pyOpenSSL).
* Now you need to download and setup [Twisted](http://twistedmatrix.com/trac/).

This is your environment to allow you to run punjab (the BOSH service) correctly.

**Step 2:** We need to now get the latest version of punjab from [github](https://github.com/twonds/punjab).  Clone or uncompress the file achieve and open a terminal window window into the directory.
Now we need to setup the service in a way were we will be able to talk to facebook.  **Note: you will require SSL certs**.  There are a number of examples of people using MD5 hash to connect to facebook and none of these worked for me.  I am not 100% sure but I think you may no longer be able to connect in this manor... you therefore require SSL keys.

Create and edit a file in the punjab folder.  I called mine "punjab.tac".  Below is the contents of my file

```
# punjab tac file
from twisted.web import server, resource, static
from twisted.application import service, internet
from twisted.internet import reactor, ssl

from punjab.httpb  import Httpb, HttpbService

root = static.File("./html")

bosh = HttpbService(1)

root.putChild('http-bind', resource.IResource(bosh))

site  = server.Site(root)

application = service.Application("punjab")
internet.TCPServer(5280, site).setServiceParent(application)

#
# RMW adding TLS support for Facebook chat/xmpp support
#
sslContext = ssl.DefaultOpenSSLContextFactory(
        'PATH_TO_KEY_FILE.key',
        'PATH_TO_CERT_FILE.crt',
)
reactor.listenSSL(
        5281,
        site,
        contextFactory=sslContext,
)

# To run this simply to twistd -y punjab.tac
# To run in debug mode twistd -n -l - -y punjab.tac
```

You should now be able to start the service with the command `twistd -y punjab.tac`

When the service starts for me I am asked for the passphrase for me private key... once I enter this I see the following output

```
2013-10-16 10:14:32-0700 [-] Log opened.
2013-10-16 10:14:32-0700 [-] twistd 13.1.0 (C:\Python27\python.exe 2.7.3) starting up.
2013-10-16 10:14:32-0700 [-] reactor class: twisted.internet.selectreactor.SelectReactor.
2013-10-16 10:14:32-0700 [-] Site starting on 5280
```

**Step 3:** Connecting to facebook with our minimal web client application.

Next you are going to need to setup your facebook application.  Follow <a href="https://developers.facebook.com/docs/opengraph/getting-started/">this guide</a> if you do not already have your facebook app setup.

Take note of both your AppId and your AppSecret.  You will require both of these in the html client.

Now you can download my sample HTML application from [github](https://github.com/coreyauger/facebook_xmpp_webclient).

You will need to edit the APP_ID and APP_SECRET in the html file.  Also you may need to change the location and/or port for your BOSH server.  Right now it is setup for http://localhost:5280/http-bind

**Note: You would not want to share your APP_SECRET in production release**

Here is a final screen shot of the simple app.
![stack](http://4.bp.blogspot.com/-mNb2ulKQEzk/UmAbS7DAcjI/AAAAAAAAE9Q/fkyhmC_IbXg/s1600/xmpp_screen.png "stack")

