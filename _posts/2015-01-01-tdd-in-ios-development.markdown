---
layout: post
title:  "TDD in iOS development [english]"
date:   2015-01-01 14:00
categories: ios tdd xcode
---

<center>*In this post I’ll show you how to start with TDD in a iOS basic project. Let's use the XCode's default unit test tool. I'll assume you're familiar with the [TDD cycle][tdd-cycle] and the terminology.*</center>

Start by launching the XCode and create a new project. Make sure that you have selected the option Include Unit Test on the screen.

![]({{ site.url }}/assets/tdd-ios/um.png)

Will be create for you a folder <yourProjectName>Tests with ours already known files (.h and .m).

![]({{ site.url }}/assets/tdd-ios/dois.png)

{% highlight objective-c %}
#import <SenTestingKit/SenTestingKit.h>

@interface unitTestIOSTests : SenTestCase
@end

@implementation unitTestIOSTests
- (void)setUp
{
}
- (void)tearDown
{
}
- (void)testExample
{
STFail(@"Unit tests are not implemented yet in unitTestIOSTests");
}
@end
{% endhighlight %}

Now, you can run (select the Test Schema on the menu) and see the fail test result on the console.

{% highlight objective-c %}
[unitTestIOSTests testExample]:
Unit tests are not implemented yet in unitTestIOSTests

Test Case '-[unitTestIOSTests testExample]' failed (0.000 seconds).
Test Suite 'unitTestIOSTests' finished at 2012-11-04 16:48:08 +0000.
Executed 1 test, with 1 failure (0 unexpected) in 0.000 (0.000) seconds
{% endhighlight %}

Like jUnit and others frameworks, the OCUnit has the setUp and tearDown method in his superclass. There are assertions methods too, like STAssertTrue, STAssertEqualObjects, etc. Now let’s start to use TDD to create something useful. First, remove the fail test from the code and create a new test method.

{% highlight objective-c %}
-(void) testUserExists {
User *user = [[User alloc] init];
STAssertNotNil(user, @"there is an user");
}
{% endhighlight %}

Run the test and you got a error. We wanna an entity User but we don’t have it yet. Let’s create it.

{% highlight objective-c %}
#import <Foundation/Foundation.h>
@interface User : NSObject {
}
@end
{% endhighlight %}

Run again and the test will to pass. Let’s add a new test. I wanna give a name and a last name to this user.

{% highlight objective-c %}
-(void) testUserExistsWithNameLastName {
User *user = [[User alloc] initWithName:@"JOAO" lastName:@"SILVA"];
STAssertNotNil(user, @"there is an user");
}
{% endhighlight %}

Run the test. Error. Now you have to add theses properties in the .h file:

{% highlight objective-c %}
@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSString *lastName;

-(id)initWithName:(NSString *)newName lastName:(NSString *)newLastName;
{% endhighlight %}

And a constructor in the .m file.

{% highlight objective-c %}
@synthesize name, lastName;

-(id)initWithName:(NSString *)newName lastName:(NSString *)newLastName {
if (self = [super init]) {
name = newName;
lastName = newLastName;
}
return self;
}
{% endhighlight %}

Run again and will to pass. Let’s continue and add another test method.

{% highlight objective-c %}
-(void) testThereIsAMessage {
Message *msg = [[Message alloc] init];
STAssertNotNil(msg, @"there is a message");
}
{% endhighlight %}

Just the same way, you have to create a new class called Message. Let’s put some properties too.

{% highlight objective-c %}
#import <Foundation/Foundation.h>
@interface Message : NSObject
@property NSString* to;
@property NSString* from;
@property NSString* body;
@end
{% endhighlight %}

Let's create a test do the Message class.

{% highlight objective-c %}
-(void) testThereIsAMessageReadyToSend {
Message *msg = [[Message alloc] init];
msg.to = @"aaa@bbb.com";
msg.from = @"zzz@yyycom";
msg.body = @"is there anybody out there?";
STAssertNotNil(msg, @"there is a message ready!");
}
{% endhighlight %}

Ok, now we can make some improvement. An User can have many messages, right? Let’s create a test for this feature before the real implementation.

{% highlight objective-c %}
-(void) testAddingMessageToUser {
Message *msg = [[Message alloc] init];
msg.to = @"aaa@bbb.com";
msg.from = @"zzz@yyycom";
msg.body = @"is there anybody out there?";

User *user = [[User alloc] initWithName:@"JOAO" lastName:@"SILVA"];
[user addMessage:msg];

STAssertEquals( [[user messageList] count], (NSUInteger)1, @"message added.");
}
{% endhighlight %}

If run now, of course will fail, like we already seen. This is the regular TDD cycle, as your know. Ok, now lets create this feature.

1. Import our Message class in the User class.
2. Add NSArray property to store the messages
3. Create a method to add the messages in this list

In the end, the User.h will be:

{% highlight objective-c %}
#import <Foundation/Foundation.h>

@class Message;

@interface User : NSObject {
}
@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSString *lastName;
@property (nonatomic, strong) NSArray *messageList;

-(id)initWithName:(NSString *)newName lastName:(NSString *)newLastName;
-(void)addMessage:(Message* )message;
{% endhighlight %}

And the implementation file:

{% highlight objective-c %}

#import "User.h"

@implementation User

@synthesize name, lastName, messageList;

-(id)initWithName:(NSString* )newName lastName:(NSString *)newLastName {
 if (self = [super init]) {
 name = newName;
 lastName = newLastName;
 messageList = [[NSArray alloc] init];
 }
 return self;
}

-(void)addMessage:(Message* )message {
 NSArray *newMessages = [messageList arrayByAddingObject:message];
 messageList = newMessages;
}

@end

{% endhighlight %}

Now you can run and check the result. Well, that is it. You can get the [full code on Github.][full-code-ios-tdd]

[full-code-ios-tdd]: https://github.com/acneto/unit-test-ios-tutorial
[tdd-cycle]: http://en.wikipedia.org/wiki/Test-driven_development
