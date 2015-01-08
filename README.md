mod_objc
========

Section: H's Computer History Museum

###mod_objc

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
```
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

###Todo

- resolve mem-leaks created by handler and command tables

###Contact

[@helje5](http://twitter.com/helje5) | helge@alwaysrightinstitute.com
