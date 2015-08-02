# Objective-C-Fake-Code
Objective-C Fake Code


## __1. Language & Runtime__

#### 1.1 Creating String Representations for Enumerated Type

```objective-c
￼NSString * const UITableViewCellStyleDescription[] = { 
	[UITableViewCellStyleDefault] = @"Default",
	[UITableViewCellStyleSubtitle] = @"Subtitle",
	[UITableViewCellStyleValue1] = @"Value 1",
	[UITableViewCellStyleValue2] = @"Value 2"
};
UITableViewCellStyle style = ...;
NSString *description = UITableViewCellStyleDescription[style];
```

#### 1.2 Adding a Property to a Category

* NSObject+AssociatedObject.h

``` objc 
@interface NSObject (AssociatedObject)
@property (nonatomic, strong) id associatedObject;
@end
```

*  NSObject+AssociatedObject.m

``` objc
@implementation NSObject (AssociatedObject)
@dynamic associatedObject;
- (void)setAssociatedObject:(id)object {
	objc_setAssociatedObject(self, @selector(associatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC); 
}
- (id)associatedObject {
	return objc_getAssociatedObject(self, @selector(associatedObject));
}
```

#### 1.3 Swizzling a Method

``` objc
#import <objc/runtime.h>
@implementation UIViewController (Tracking)

+ (void)load {
	static dispatch_once_t onceToken; dispatch_once(&onceToken, ^{
		Class class = [self class];
		// When swizzling a class method, use the following:
		// Class class = object_getClass((id)self);
		SEL originalSelector = @selector(viewWillAppear:);
		SEL swizzledSelector = @selector(xxx_viewWillAppear:);
		Method originalMethod = class_getInstanceMethod(class, originalSelector);
		Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
		BOOL didAddMethod = class_addMethod(class,
		                                    originalSelector, 	method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
		if (didAddMethod) {
		    class_replaceMethod(class,
		                        swizzledSelector, method_getImplementation(originalMethod), 	method_getTypeEncoding(originalMethod));
		}
		else {
		    method_exchangeImplementations(originalMethod, swizzledMethod);
		}
	});
}

#pragma mark - Method Swizzling
- (void)xxx_viewWillAppear:(BOOL)animated {
	[self xxx_viewWillAppear:animated]; NSLog(@"viewWillAppear: %@", self);
}
```

#### 1.4 Determining the Type of a Property

``` objc
const char *attributes = property_getAttributes(class_getProperty([self  class], sel_getName(@selector(property))));
NSString *typeAttribute = [[[NSString stringWithUTF8String:attributes]  componentsSeparatedByString:@","] firstObject];
const char *propertyType = [[typeAttribute substringFromIndex:1] UTF8String];
if (strcmp(propertyType, @encode(float)) == 0) {
        // float
} else if (strcmp(propertyType, @encode(int)) == 0) {
        // int
}
```

#### 1.5 Specifying the Availability of a Method

``` objc
void foo() __attribute__((availability(macosx, introduced=10.4,
                                           deprecated=10.6,
                                           obsoleted=10.7)));
```

#### 1.6 Hiding a Class

``` objc
__attribute__((visibility("hidden"))
@interface HiddenClass : Superclass // ...
@end
```

#### 1.7 Hiding a Method

``` objc
- (BOOL)respondsToSelector:(SEL)selector {
    if (selector == @selector(methodToHide)) {
        return NO; }
    return [[self class] instancesRespondToSelector:selector];
}
```

#### 1.8 Ignoring Compiler Warnings

``` objc
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wgnu" id object = object ?: [NSNull null];
#pragma clang diagnostic pop
```

#### 1.9 Determining the Current System Memory Usage

``` objc
#import <mach/mach.h>
#import <sys/sysctl.h>
vm_statistics_data_t vmStats;
mach_msg_type_number_t infoCount = HOST_VM_INFO_COUNT;
kern_return_t status = host_statistics(mach_host_self(), HOST_VM_INFO, (host_info_t)&vmStats, &infoCount);
natural_t memoryUsage = 0; // in bytes
if (status == KERN_SUCCESS) {
    memoryUsage = vmStats.wire_count * 1024.0;
}
```

#### 1.10 Creating Variadic Method

``` objc
- (void)method:(id)object, ... NS_REQUIRES_NIL_TERMINATION { va_list args;
    va_start(args, object);
    while (object) {
        // ...
        object = va_arg(args, id);
    }
    va_end(args);
}
```

#### 1.11 Creating a Variadic Function 

``` objc
static double average(int count, ...) {
    va_list args;
    va_start(args, count);
    int sum = 0;
    for (int i = 0; i < count; i++) {
        sum += va_arg(args, int);
    }
    va_end(args);
    return (double)sum / (double)count;
}
double avg = average(4, 2, 3, 4, 5);
```

#### 1.12 Overloading Functions

``` objc
#include <math.h>
__attribute__((overloadable)) CGFloat CGFloat_floor(double d) {
    return (CGFloat)floor(d);
}
__attribute__((overloadable)) CGFloat CGFloat_floor(float f) {
    return (CGFloat)floorf(f);
}

float __attribute__((overloadable)) tgsin(float x) {
    return sinf(x);
}
double __attribute__((overloadable)) tgsin(double x) {
    return sin(x);
}
long double __attribute__((overloadable)) tgsin(long double x) {
    return sinl(x);
}
```

#### 1.13 Conditionally Compiling for iOS & OS X Targets


``` objc
#import <Availability.h>
id color = nil;
#if defined(__MAC_OS_X_VERSION_MAX_ALLOWED)
color = [NSColor orangeColor];
#elsif defined(__IPHONE_OS_VERSION_MIN_REQUIRED)
color = [UIColor purpleColor];
#endif
```

#### 1.14 Requiring Method to call super

``` objc
- (void)method __attribute__((objc_requires_super));
```

#### 1.15 Determining the Caller of a Method

| Stack | Framework | Address| Class | Function | Line |
| ----- |:---------:| ------:| -----:| --------:| ----:|
| 1     |   UIKit   | 0x004f | UIKit | Function |  32 

``` objc
NSString *callerSymbol = [NSThread callStackSymbols][1];
NSCharacterSet *characterSet = [NSCharacterSet characterSetWithCharactersInString:@" +,-.?[]"];
NSArray *components = [callerSymbol componentsSeparatedByCharactersInSet:characterSet];
components = [components filteredArrayUsingPredicate:[NSPredicate predicateWithFormat:@"self <> ’’"]];
NSString *stack = components[0];
NSString *framework = components[1];
NSString *address = components[2];
NSString *classCaller = components[3];
NSString *functionCaller = components[4];
NSString *lineCaller = components[5];
```

#### 1.16 Intentionally Crashing the Current Process

``` objc
__builtin_trap();
```

---

## __2. Grand Central Dispatch__

#### 2.1 Benchmarking the Execution Time of an operation

``` objc
extern uint64_t dispatch_benchmark(size_t count, void (^block)(void))...----
size_t const objectCount = 1000;
uint64_t t = dispatch_benchmark(10000, ^{
    @autoreleasepool { id obj = @42;
            NSMutableArray *array = [NSMutableArray array]; for (size_t i = 0; i < objectCount; ++i) {
                [array addObject:obj];
            }
	}
});
NSLog(@"-[NSMutableArray addObject:] : %llu ns", t);
```


#### 2.2 Dispatching a Timer

``` objc
dispatch_queue_t queue = dispatch_queue_create(NULL, DISPATCH_QUEUE_CONCURRENT);
dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
int64_t delay = 30 * NSEC_PER_SEC;
int64_t leeway = 5 * NSEC_PER_SEC;
dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, delay , leeway);
dispatch_source_set_event_handler(timer, ^{
    NSLog(@"Ding Dong!");
});
dispatch_resume(timer);
```

#### 2.3 Monitoring Local File Changes

``` objc
NSURL *fileURL = [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory
       inDomains:NSUserDomainMask] firstObject];
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
int fileDescriptor = open([fileURL fileSystemRepresentation], O_EVTONLY);
unsigned long mask = DISPATCH_VNODE_EXTEND | DISPATCH_VNODE_WRITE | DISPATCH_VNODE_DELETE;
__block dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_VNODE, fileDescriptor,mask,queue);
dispatch_source_set_event_handler(source, ^{
dispatch_source_vnode_flags_t flags = dispatch_source_get_data(source);
if (flags) {
    dispatch_source_cancel(source);
    dispatch_async(dispatch_get_main_queue(), ^{
    // ...
       });
    }
});
dispatch_source_set_cancel_handler(source, ^{
    close(fileDescriptor);
});
dispatch_resume(source);
```

---

## __3. Random__

#### 3.1 Creating a Random Integer

``` objc
// Random int between 0 and N - 1
NSUInteger r = arc4random_uniform(N);
// Random int between 1 and N
NSUInteger r = arc4random_uniform(N) + 1;
```

#### 3.2 Creating a Random Double

``` objc
srand48(time(0));
double r = drand48();

// Random Color
srand48(time(0));
UIColor *color = [UIColor colorWithRed:drand48()
                                   green:drand48()
                                    blue:drand48()
                                  alpha:1.0f];
```

#### 3.3 Creating a Random String

``` objc
NSString *letter = [NSString stringWithFormat:@"%c", arc4random_uniform(26) + ’a’];
```

#### 3.4 Creating a Random Date

``` objc
NSTimeInterval timeInterval = (NSTimeInterval)arc4random_uniform(pow(2.0, 32.0) - 1.0);
NSDate *date = [NSDate dateWithTimeIntervalSinceReferenceDate:timeInterval];
```

---

## __4. Collections__

#### 4.1 Accessing Mutable Dictionary in a Thread-Safe Manner

``` objc
@property NSMutableDictionary *mutableDictionary;
@property dispatch_queue_t queue;
- (void)setObject:(id)object
           forKey:(id)key{
     dispatch_barrier_async(self.queue, ^{
        self.mutableDictionary[key] = object;
    });
}
```

#### 4.2 Reversing an Array

``` objc
NSArray *array = ...;
NSArray *reversed = [[array reverseObjectEnumerator] allObjects];
```

#### 4.3 Filtering Objects in Array by Class

``` objc
NSArray *mixedArray = @[@"a", @"b", @"c", @(1), @(2), @(3)];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"self isKindOfClass: %@", [NSString class]];
NSArray *letters = [mixedArray filteredArrayUsingPredicate:predicate];
```

#### 4.4 Computing the Sum of an Array

``` objc
NSArray *array = @[@1, @2, @3];
NSNumber *sum = [array valueForKeyPath:@"@sum.self"];
```

#### 4.5 Removing Duplicate Objects from an Array

``` objc
NSArray *array = @[@"a", @"b", @"c", @"a", @"d"];
NSArray *uniqueArray = [array valueForKeyPath:@"@distinctUnionOfObjects.self"];
```

``` objc
NSSet *set = [NSSet setWithArray:array]; 
NSArray *uniqueArray = [set allObjects];
```

``` objc
NSOrderedSet *orderedSet = [NSOrderedSet orderedSetWithArray:array];
NSArray *uniqueArray = [orderedSet array];
```

---

## __5. Cartography__

#### 5.1 Converting Degrees to Radians

``` objc
double radians = degrees * M_PI / 180.0f;
```

#### 5.2 Converting Radians to Degrees

``` objc
double degrees = radians * 180.0f / M_PI;
```

#### 5.3 Convert Radians to CLLocationDirection

``` objc
CLLocationDirection direction = fmod(degrees, 360.0f) + 90.0f;
```

---

## __6. Graphics__

#### 6.1 Animating a CAGradientLayer

``` objc
NSArray *colors = @[(id)[[UIColor redColor] CGColor], (id)[[UIColor orangeColor] CGColor]];
NSArray *locations = @[@(0.0), @(1.0)]; NSTimeInterval duration = 1.0f;
[UIView animateWithDuration:duration animations:^{
    [CATransaction begin];
    {
        [CATransaction setAnimationDuration:duration];
        [CATransaction setAnimationTimingFunction:[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut]];
        [(CAGradientLayer *)self.layer setColors:colors];
        [(CAGradientLayer *)self.layer setLocations:locations];
    }
    [CATransaction commit];
}];

+ (Class)layerClass {
    return [CAGradientLayer class];
}

#pragma mark - CALayerDelegate
- (id <CAAction>)actionForLayer:(CALayer *)layer forKey:(NSString *)event{
    
    id <CAAction> action = [super actionForLayer:layer forKey:event];
    if ((!action || [(id)action isEqual:[NSNull null]]) && [event isEqualToString:@"colors"]) {
        action = [CABasicAnimation animationWithKeyPath:event];
    }
    return action;
    
}
```

#### 6.2 Creating an Image for a Swatch of Color

``` objc
static UIImage * UIImageForSwatchOfColorWithSize(UIColor *color, CGSize size)
{
    UIImage *image = nil;
    CGRect rect = CGRectMake(0.0f, 0.0f, size.width, size.height);
    UIGraphicsBeginImageContext(rect.size); {
        CGContextRef c = UIGraphicsGetCurrentContext();
        CGContextSetFillColorWithColor(c, [color CGColor]); CGContextFillRect(c, rect);
        image = UIGraphicsGetImageFromCurrentImageContext(); }
    UIGraphicsEndImageContext();
    return image;
}
```

#### 6.3 Getting Color RGB Components

``` objc
UIColor *color = ...;
CGFloat r, g, b, a;
[color getRed:&r green:&g blue:&b alpha:&a];
```

---

## __7. UIKit__

#### 7.1 Creating a Snapshot of a View

``` objc
UIView *view = ...;
double scale = [[UIScreen mainScreen] scale];
UIImage *snapshot = nil;
UIGraphicsBeginImageContextWithOptions(view.bounds.size, NO, scale); {
    if ([self respondsToSelector:@selector(drawViewHierarchyInRect: afterScreenUpdates:)]) {
          [self drawViewHierarchyInRect:view.bounds afterScreenUpdates:YES];
    } else {
          [self.layer renderInContext:UIGraphicsGetCurrentContext()];
    }
    snapshot = UIGraphicsGetImageFromCurrentImageContext();
}
UIGraphicsEndImageContext();
```

#### 7.2 Determining if UIViewController is Visible

``` objc
- (BOOL)isVisible {
	return [self isViewLoaded] && self.view.window;
}
```

#### 7.3 Removing Drop Shadow from UIWebView

``` objc
for (UIView *view in [[[webView subviews] objectAtIndex:0] subviews]) {
    if ([view isKindOfClass:[UIImageView class]] && view.frame.size.width == 1.0f) {
              view.hidden = YES;
        }
}
```

#### 7.4 Making a Device Vibrate

``` objc
@import AudioToolbox;
// Plays an alert noise if vibration not supported on device
AudioServicesPlayAlertSound(kSystemSoundID_Vibrate);

// No-op on devices that do not support vibration
AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);
```

