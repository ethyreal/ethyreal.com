---
title:  "Asterisk Syntax Lesson Learned"
date:   2010-11-03 09:43:00 -0800
categories: asterisk pbx
---
Over the last few weeks I have converted an old Digilogic Vox IVR system over to use Asterisk. During this process I have discovery quite a few oddities in the Asterisk dial plan logic.


```
exten => 1,2,Set(SOME_VAR=2112)
```

is not the same as:

```
exten => 1,2,Set( SOME_VAR = 2112)
```

In fact almost not white space is allow at all, with exception of the expression syntax.

```
exten => 1,1,GotoIf($[ ${SOME_VAR} = 2112 ]?context_if_true:context_if_false)
```

Also in ```GotoIf()``` and ```GosubIf()``` statements either branch is optional. So:


```
exten => 1,1,GotoIf($[ expression.. ]?true_only:)
```

or:

```
exten => 1,1,GotoIf($[ expression.. ]?:false_only)
```

Whichever statement you don't catch just falls through to the next line in current dial plan context.

The boolean "and" operation is the most distrubing. Lets say we have three variables:

```
exten => s,1,NoOp()
exten => s,n,Set(VAR_ONE=1)
exten => s,n,Set(VAR_TWO=1)
exten => s,n,Set(VAR_THREE=1)
```

All three variables are set to 1. Then we want to test combinations of them. I chose a false only fall through path as follows:

```
; here if VAR_ONE = 1 AND VAR_THREE = 0 the print "true" otherwise if false jump to test_two
exten => s,n,GotoIf($[${VAR_ONE}=1] & $[${VAR_THREE}=0]?:test_two)
exten => s,n,NoOp(true) ;this should not print because VAR_THREE !=0	

; here in test two if  VAR_TWO = 1 AND VAR_THREE = 1 then print true, self move on to test_three 
exten => s,n(test_two),GotoIf($[${VAR_TWO}=1] & $[${VAR_THREE}=1]?:test_three)
exten => s,n,NoOp(true) ; this should print! 

; Now lets try reversing the order of testing:

; now we test VAR_THREE = 0 first! then test VAR_ONE
exten => s,n(test_three),GotoIf($[${VAR_THREE}=0] & $[${VAR_ONE}=1]?:test_four)
exten => s,n,NoOp(true) ;again should not print.

; last we revers the second test if THREE=1 & TWO=1
exten => s,n(test_four),GotoIf($[${VAR_THREE}=1] & $[${VAR_TWO}=1]?:test_done)
exten => s,n,NoOp(true) ; this should print! 
exten => s,n(test_done),NoOp()
```

The results are: Test one prints TRUE! and test two prints TRUE. But when I reverse them I receive the expected result: Test three doesn't print, and test four does print true.

I think I undestand it as the "and" operator returns the value of the first statement? this seems to happen even if the second statement if false. That or purhaps the second statement is not running at all, and it is only evaluating the first block. The odd thing is if that were true you would think the system would crash. Every other invalid operation crashes the system, if this expression was not executing why would it not crash where the others did. Maybe I'm more confused now than when I started.

Right or not I chose to do two if statements and abstract the mess into a ```Gosub()```, while this isn't ideal at least I have results I can count on.