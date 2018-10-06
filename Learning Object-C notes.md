	// Import the foundation framework.
	// This includes things like the NSObject, NSString and NSNumber classes
	#import <Foundation/Foundation.h>
	
	// DWKRandomController is a subclass of NSObject
	@interface DWKRandomController : NSObject
	{
	    IBOutlet NSTextField *textField;
	}

	    - (IBAction)generate:(id)sender;
	    - (IBAction)seed:(id)sender;

	@end
