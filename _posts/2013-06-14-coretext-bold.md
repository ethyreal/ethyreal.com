---
title:  "CoreText Bold"
date:   2013-06-14 09:43:00 -0800
categories: ios fonts coretext
---
Simple text editor paradigm, user selects a font from a menu/list and you want to know if they have a bold font or not, so you can show them in the UI. Simply check the symbolic traits on the CTFontRef:

```swift
-(BOOL)isBoldFont:(CTFontRef)font
{
    NSDictionary *traits = (__bridge_transfer NSDictionary *)CTFontCopyTraits(font);
 
    BOOL hasBoldTrait = ([[traits objectForKey:(id)kCTFontSymbolicTrait] unsignedIntValue] & kCTFontBoldTrait) == kCTFontBoldTrait;
 
    NSString *styleAttribute = (__bridge_transfer NSString *)CTFontCopyAttribute(font, (CFStringRef)kCTFontStyleNameAttribute);
 
    if (hasBoldTrait || [styleAttribute isEqualToString:@"Bold"]) {
        
        return YES;
    }
    
    return NO;
}
```

If you want to change it to bold you can create a new font by copying symbolic traits:

```swift
-(CTFontRef)createBoldFontFromFont:(CTFontRef)fromFont
{
     CTFontCreateCopyWithSymbolicTraits(fromFont, 0.0, NULL, kCTFontBoldTrait, kCTFontBoldTrait);
 
}
```

This will return NULL if the font family does not have a bold font. You could use this to check the availability of bold or just wether or not you actually created a bold font.

If you want to force a font to render "bold" even though there is no actual bold font face you could change the text drawing mode in your draw rect:

```swift
CGContextRef context = UIGraphicsGetCurrentContext();
CGContextSetTextDrawingMode(context, kCGTextFillStroke);
```

That will stroke and fill the glyphs instead of just filling them ( kCGTextFill ) which I believe is the default.
