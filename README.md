mod_objc
========

Section: H's Computer History Museum

### mod_objc

This is an Apache 1 (yes, *one*, like Apache 1.3) module I played with back in
2001. Being an Apache 1 module it is pretty much useless today.

The idea was that instead of writing Apache modules in C, you would write them
as an Objective-C (GNUstep or libFoundation) bundle. The principal class of the
bundle would act as the module entry point and could be used to respond to
requests and such. Basically anything you could do using a regular C Apache
module.

One hack/trick in this was that Apache unloads the modules during the
configuration phase. This didn't (doesn't) play well with the GNU Objective-C
runtime. So we had to hang on to the bundle (keep a reference to the shared
library alive).

This was never used in production for anything. And you shouldn't use it
either :-) Probably doesn't even compile anymore.

### Sample

Module:
```Objective-C
#include "ApacheModule.h"
#import <Foundation/Foundation.h>

@interface ApTest : ApacheModule
@end

@implementation ApTest

- (int)handleTextHtmlRequest:(ApacheRequest *)_rq {
  printf("%s ...\n", __PRETTY_FUNCTION__);
  return ApacheDeclineRequest;
}

@end
```

GNUstep Makefile:
```Make
include $(GNUSTEP_MAKEFILES)/common.make

BUNDLE_NAME = ApTest

ApTest_PRINCIPAL_CLASS = ApTest
ApTest_OBJC_FILES = \
	ApTest.m			\
	ApTest_module_structure.m	\

ApTest_BUNDLE_LIBS   += -lApacheAPI

include $(GNUSTEP_MAKEFILES)/bundle.make

before-ApTest-all:: 
	genApacheModule.sh ApTest ApTest_module_structure.m
```

Apache Config:

    # Setup
    
    LoadModule gnustep_conf_module \
            Libraries/ix86/linux-gnu/apache/mod_gnustep_conf.so
    LoadModule gsbundle_module \
            Libraries/ix86/linux-gnu/apache/mod_gsbundle.so
    AddModule  mod_gsbundle.m
    
    # Load Bundle
    
    LoadBundle   ApTest.bundle
    
    <Location "/AlwaysRight/">
      SetHandler ap-test
    </Location>

### Todo

- resolve mem-leaks created by handler and command tables


ApacheWO
========

Also included is ApacheWO - a bundle written using mod_objc.
This used the SOPE application framework to host WO like applications directly
from within Apache 1. Again, just a toy, but a pretty cool one ;-)

Essentially you could just drop in your components into the Apache htdocs
directory, like

    LoadApacheBundle  ApacheWO.apache

    htdocs/HelloWorld.wo/
    
    # .html
    <WEBOBJECT NAME="Frame">
      .wo based page
    
      a: <WEBOBJECT NAME="a"></WEBOBJECT><br />
      b: <WEBOBJECT NAME="b"></WEBOBJECT><br />
      c: <WEBOBJECT NAME="AddAB"></WEBOBJECT><br />
    
      Action Link to Page2: <WEBOBJECT NAME="Page2">Page2</WEBOBJECT>
    </WEBOBJECT>
    
    # .wod    
    Frame: Frame    { title = name;  }
    AddAB: WOString { value = addAB; }
    a:     WOString { value = a;     }
    b:     WOString { value = b;     }
    Page2: WOHyperlink { action = gotoPage2; }

You can embed other components etc.

Or you have a custom WORequestHandler? Just drop it into your htdocs like that:

    AddType skyrix/request-handler .rqh
    
    File xmlrpc.rqh:
      { class = "XmlRpcRequestHandler"; }

But wait, now it get's fancy. Since those are just standard Apache 1 objects,
you can even embed components within other Apache engines, like SSI (Server Side
Includes):

    <html>
      <head><title>Server Side Include</title></head>
    
      <body>
        <h3>Server Side Include</h3>
    
        Today is <!--#echo var="DATE_LOCAL" --><br /><br />
        Inluded .wox page:
        <table border="1" width="100%"><tr><td>
        <!--#include virtual="Page2.wox" -->
        </td></tr></table>
      </body>
    </html>

### Contact

[@helje5](http://twitter.com/helje5) | helge@alwaysrightinstitute.com
