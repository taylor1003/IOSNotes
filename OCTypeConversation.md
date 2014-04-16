Objective-C 类型转换
===================

NSNunmber转NSString
-------------------

	NSNumber *A = @"45";
	NSString *B;
	NSNumberFormatter* numberFormatter = [[NSNumberFormatter alloc] init];
	B = [numberFormatter stringFromNumber:A];
	[numberFormatter release];

NSString与int之间的转换
---------------------

NSString *tempA = @"123";

NSString *tempB = @"456";

### 1，字符串拼接

NSString *newString = [NSString stringWithFormat:@"%@%@",tempA,tempB];

### 2，字符转int

int intString = [newString intValue];

### 3，int转字符

NSString *stringInt = [NSString stringWithFormat:@"%d",intString];

### 4，字符转float

float floatString = [newString floatValue];

### 5，float转字符

NSString *stringFloat = [NSString stringWithFormat:@"%f",intString];


NSString与NSData之间的转换 
------------------------

### 1，NSString转NSData

NSData* data = [@"data for test" dataUsingEncoding:NSUTF8StringEncoding]; 

### 2，NSData转NSString

NSString *result = [[NSString alloc] initWithData:data  encoding:NSUTF8StringEncoding];
 
NSData与char*之间的转换
---------------------

### 1，NSData转char*

NSData *data; 

char *test=[data bytes]; 

### 2，char* 转NSData对象 

byte* tempData = malloc(sizeof(byte)*16); 

NSData *content=[NSData dataWithBytes:tempData length:16];