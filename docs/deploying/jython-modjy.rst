.. _deploying-jython-modjy:

Jython and modjy
================

flask runs happily on the JVM using jython using the `modjy`_ Servlet/WSGI
gateway.  The nice thing is that modjy is already included in a standard
jython installation.

.. note:: The author only has experience using the combination flask, jython,
   tomcat.  If you use another servlet container, you're on your own.

To actually use flask with Jython, you need:

- decide on where you actually want to *deploy* your application
  to -- i.e. which *directory* inside your *tomcat* installation

- decide on the *URL* you want to serve your application from

- configure your tomcat such that it knows how to serve requests such
  that tomcat routes requests to your application through the `modjy`
  gateway

- package up and *deploy* your applictaion

Configuring modjy
-----------------

To *configure* your application to be served by tomcat, you need to
add the `modjy` gateway servlet to your web app's `web.xml` directory.

Directory Layout
~~~~~~~~~~~~~~~~

So, suppose your tomcat web app root is `$TOMCAT/webapps` and you want to
deploy a your application to `$TOMCAT/webapps/flaskapp`, you need to create
a structure like::

    flaskapp
    ├── WEB-INF
    │   ├── lib
    │   │   ├── jython.jar
    │   │   └── readme.txt
    │   ├── lib-python
    │   │   └── readme.txt
    │   └── web.xml
    ├── demo_app.py
    └── readme.txt

The directories have the following roles:

**WEB-INF**
    This is the web-app configuration directory.  It holds the `web.xml`
    configuration file (see below) and run-time libraries.

**lib**
    This directory contains *Java Packages* needed by your web app -- most
    notably the *jython.jar*

**lib-python**
    This directory may contain additional *python* packages used by your
    application.

The flask app in `demo_app.py` looks something like this:


.. sourcecode:: python

    from flask import Flask

    app = Flask(__name__)


    @app.route('/')
    def hello_world():
        return 'Hello World!'


    def handler(environ, start_response):
        return app.wsgi_app(environ, start_response)


Note that we define a WSGI callable `handler`.  This callable will be called
by the `modjy` gateway.

Configuring
~~~~~~~~~~~

Having the above structure in place, we now need to tell tomcat what web
application we have and how it should route requests to it.  This is done
using the `web.xml` file.  Below is a working minimal example::

    <?xml version="1.0" encoding="iso-8859-1"?>
    <!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">
    <web-app>
      <display-name>modjy demo application</display-name>
      <description>modjy WSGI demo application</description>
    
      <servlet>
        <servlet-name>modjy</servlet-name>
        <servlet-class>com.xhaus.modjy.ModjyJServlet</servlet-class>
        <init-param>
          <param-name>python.home</param-name>
          <param-value>
          /usr/local/Cellar/jython/2.5.3b1/libexec</param-value>
        </init-param>
        <init-param>
          <param-name>app_filename</param-name>
          <param-value>demo_app.py</param-value>
        </init-param>
        <init-param>
          <param-name>app_callable_name</param-name>
          <param-value>handler</param-value>
        </init-param>
        <init-param>
          <param-name>load_site_packages</param-name>
          <param-value>1</param-value>
        </init-param>
        <init-param>
          <param-name>log_level</param-name>
          <param-value>debug</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
      </servlet>

      <servlet-mapping>
        <servlet-name>modjy</servlet-name>
        <url-pattern>/*</url-pattern>
      </servlet-mapping>

    </web-app>

For all the available `init-param` options, please have a look at
http://opensource.xhaus.com/projects/modjy/wiki/ModjyConfiguration.

A few things to notice:

- We need to define the *JYTHON_HOME* using the `python.home` parameter.  This
  is needed because `jython` needs to find your `site-packages`.  For example,
  `flask` here is installed globally in this jython home.  You **can** use
  `virtualenv` with jython, just set the `python.home` to your virtualenv
  root.

- We need to tell the gateway the *name* of the `demo_app.py` file and a
  *callable name*.  This is used by `modjy` to resolve the WSGI callable it
  needs to call for a request.

- using the `servlet-mapping` tag, we're telling tomcat, that we'd like to
  route all (`/*`) requests to the `modjy` servlet.  Note that you **could**
  define multiple `servlet` tags, and have a `servlet-mapping` for each one.


**XXX**
   - note about memory leaks in multithread mode
   - note about failing to get templates going

Packaging and Deploying
-----------------------

**TBD**

- test war file generation and upload/deploy using tomcat 7

Development Tips
----------------

**TBD**

- mention how modjy parses pth files in lib-python and how to use this
  during development


Caveats
-------

**TBD**

- mention that modjy caches callables
- mention memory leaks

.. _modjy: http://opensource.xhaus.com/projects/show/modjy

..  vim: set ft=rst ts=4 sw=4 spelllang=en expandtab tw=78 : 
