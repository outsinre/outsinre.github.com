---
layout: post
title: Run VBA Macros on Protected Worksheet UserInterfaceOnly
---

If you run macros on a protected worksheet which attempt to make changes in the worksheet, you will encounter an error:

>Run-time error '1004': Application-defined or object-defined error

# Unprotect & Protect
One option is to unprotect the worksheet, run the code / macro, and then protect it again, as shown below:

{% highlight vb.net linenos %}
Sheet1.Unprotect Password:="abc" 
 
'Enter Code / Macro
 
Sheet1.Protect Password:="abc"
{% endhighlight %} 
 
However, this method has some shortcomings:

1. If the code encounters an error or gets interrupted, your worksheet will remain unprotected;
2. Your worksheet protection password will be displayed in the vba code (vba projects usually protect worksheet with vba code instead of directly from worksheet menus), unless you protect your VBAProject (in the VB Editor code window) to disallow access or viewing of your code;
  * The *Fund Stats.xls* indeed is protected in VB Editor code window by password.
3. You will need to enter this code (ie. unprotect & protect statements) repeatedly in each macro.

# Error Handler
To overcome the first shortcoming wherein the worksheet remains unprotected upon the code encountering an error, you can use an ErrorHandler, as shown below:

{% highlight vb.net linenos %}
Sub macroProtect1()

Sheet1.Unprotect Password:="abc" 
 
	'Enable error-handling routine for any run-time error
	On Error GoTo ErrHandler   
 
	'this code will run irrespective of an error or Error Handler
	Sheet1.Cells(1, 1) = UCase("hello")

	'this code will give a run-time error, because of division by zero. The worksheet will remain unprotected in the absence of an Error Handler. 
	Sheet1.Cells(2, 1) = 5 / 0

	'this code will not run, because on encountering the above error, you go directly to the Error Handler 
	Sheet1.Cells(3, 1) = Application.Max(24, 112, 66, 4)

Sheet1.Protect Password:="abc" 

ErrHandler:
  Sheet1.Protect Password:="abc" 

End Sub 

Sub macroProtect2()

Sheet1.Unprotect Password:="abc"
 
	'skip all run-time errors
	On Error Resume Next  
 
	'this code will run irrespective of an error or Error Handler
	Sheet1.Cells(1, 1) = LCase("HELLO")

	'this code will give a run-time error, because of division by zero. The worksheet will remain unprotected in the absence of an Error Handler. 
	Sheet1.Cells(2, 1) = 5 / 0

	'this code will run, because on encountering the above error, the code continues execution from next statement, and worksheet remains protected. 
	Sheet1.Cells(3, 1) = Application.Min(24, 112, 66, 4)

Sheet1.Protect Password:="abc"
 
'Turn off error trapping and re-allow run time errors
On Error GoTo 0  

End Sub

{% endhighlight %}

The two error handlers are just examples. I think the first one is better.

### On Error Statements explained:
**On Error Resume Next**: Specifies that when a run-time error occurs, control goes to the statement immediately following the statement where the error occurred, and execution continues from that point.  
**The On Error GoTo 0**: statement turns off error trapping.  It disables enabled error handler in the current procedure and resets it to Nothing.  
**On Error GoTo Line**: Enables the error-handling routine that starts at the specified Line. The On Error GoTo statement traps all errors, regardless of the exception class.

# UserInterfaceOnly parameter
A better way to run macros in a protected worksheet would be to use the UserInterfaceOnly argument in the Protect method, by setting the UserInterfaceOnly argument to True, in the manner: `Sheet1.Protect Password:="abc" , UserInterFaceOnly:=True`.
 
UserInterFaceOnly argument is an optional argument in the Protect method and its *default is False*. Setting the UserInterfaceOnly argument to `True` means that the worksheet protection applies **only to the user interface and does not apply to macros** and this will allow Excel to run all macros in the worksheet. If this argument is omitted, protection applies both to macros and to the user interface. You can use the UserInterfaceOnly argument in a worksheet, at the beginning of the macro, to enable the user interface protection each time the macro is run. See following examples:

### Using the UserInterfaceOnly argument, in a worksheet:
{% highlight vb.net linenos %}
Sub macroProtect3()

Sheet1.Protect Password:="abc" , UserInterFaceOnly:=True
 
	'enter code
	Sheet1.Cells(1, 1) = UCase("hello")

End Sub
{% endhighlight %}

It may be noted that if you apply the Protect method with the UserInterfaceOnly argument set to True to a worksheet and then save the workbook, the entire worksheet (not just the interface) will be fully protected when you reopen the workbook. To re-enable the user interface protection after the workbook is opened, you must again apply the Protect method with UserInterfaceOnly set to True. How to solve this issue?
 
To re-enable the user interface protection after the workbook is opened, you can use the **UserInterfaceOnly argument in the Workbook_Open Event** wherein it gets enabled in all or the specified worksheets, each time the workbook is opened.

### Using the UserInterfaceOnly argument, in the Workbook_Open event:
 
**If all worksheets use the same password**, then set the UserInterfaceOnly argument as true in the Workbook_Open event, as follows. Please note that workbook events code must be placed in the code module for the **ThisWorkbook object**.

{% highlight vb.net linenos %}
Private Sub Workbook_Open()

Dim ws As Worksheet

    For Each ws In ThisWorkbook.Worksheets
    	ws.Protect Password:="abc", Contents:=True,  UserInterFaceOnly:=True
    Next ws
 
End Sub
{% endhighlight %}

**To enable user interface protection for specified worksheets only** or if the worksheets have different passwords, set the UserInterfaceOnly argument as true in the Workbook_Open event, as follows.

{% highlight vb.net linenos %}
Private Sub Workbook_Open()

	'enter below statement for each worksheet
	Sheet1.Protect Password:="abc" , UserInterFaceOnly:=True

	Sheet2.Protect Password:="def" , UserInterFaceOnly:=True

End Sub
{% endhighlight %}

# Run macro on protected worksheet, code remaining visible & editable, but protection password hidden:
 
If you want to run a macro on a protected worksheet, and keep the code visible and editable in the VB editor ie. not to protect the VBA project in the code window with a password; and further you **do not want to disclose or display the password** (used for worksheet protection) to the user, use the following method.
 
If you are using 3 worksheets (say, Sheet1, Sheet2 & Sheet3) in a workbook which are password protected and running macros, insert an additional worksheet (Sheet4).
 
## Step 1: Make the Sheet4 **very hidden** with this code:

{% highlight vb.net linenos %}
Sub hideSheet()
'you can make a worksheet "very hidden" which means that it will not be visible to the user and it can be unhidden only with a VBA code because it does not appear in the unhide list in the interface. A sheet can be made "very hidden" by a single line vba code, or through the VB Editor by selecting the relevant sheet and go to Properties Window: Properties -> Visible -> 2-xlSheetVeryHidden (select).

Worksheets("Sheet4").Visible = xlVeryHidden
    
End Sub
{% endhighlight %}
 
## Step 2: VBA code to unhide (if & when required) the "very hidden" Sheet4:

{% highlight vb.net linenos %}
Sub unhideSheet()
'you can enter the password, to unhide this "very hidden" worksheet, in this worksheet itself, which means that the password will need to be remembered separately and will not be visible or accessible till the sheet is unhidden.
'In this "very hidden" worksheet, you can store passwords which protect other worksheets on which you wish to run the macros, and these macros will call the stored password(s) from the "very hidden" worksheet.

Dim pswd As String

    pswd = Cells(1, 1)
    mypass = pswd

    pswdMatch = InputBox("Enter password to unhide sheet")

    If pswdMatch = pswd Then
        Worksheets("Sheet4").Visible = True
    Else
	Exit Sub
    End If

End Sub
{% endhighlight %}
 
## Step 3: Run macro on Sheet2, which is password protected:

{% highlight vb.net linenos %}
Sub macroProtect()

Dim pswdSheet2 As String
 
'store password to protect Sheet2, in Sheet4.Cells(2, 1)
pswdSheet2 = Sheet4.Cells(2, 1)

	   Sheet2.Protect Password:=pswdSheet2, UserInterFaceOnly:=True

	   'enter code
	   Sheet2.Cells(1, 1) = UCase("hello")

End Sub
{% endhighlight %}
 
 
The 3 steps above will ensure that you run the macro on a protected worksheet (Sheet2), whose password is hidden in Sheet4. And Sheet4 is itself protected and "very hidden", with its own password too entered & hidden in itself. You will just need to separately remember or note the password of Sheet4 to unhide as per Step2.
 
# Using the UserInterfaceOnly argument to Enable Auto Filter:

{% highlight vb.net linenos %}
Sub enableFilterOnProtectedWs1()
'Using UserInterfaceOnly argument with EnableAutoFilter property

With Sheet1
 
'this enables AutoFilter arrows when user-interface-only protection is turned on. This EnableAutoFilter property is not saved with the worksheet or session.
'if you cannot use the code to put AutoFiltering on or off (viz: .AutoFilterMode = False) when user-interface-only protection is turned on, use this EnableAutoFilter property.
.EnableAutoFilter = True

'set userinterfaceonly to True, to protect the user interface, but not macros. If this argument is omitted, protection applies both to macros and to the user interface.
.Protect Password:="123" , contents:=True, UserInterfaceOnly:=True

'removes any existing Autofilter
.AutoFilterMode = False

'AutoFilter arrows get visible, with a filter criteria
.Range("A1:B10" ).AutoFilter Field:=1, Criteria1:= "<35" 
 
End With

End Sub


Sub enableFilterOnProtectedWs2()
'enabling AutoFilter arrows when user-interface-only protection is turned on, without using the EnableAutoFilter property.

With Sheet1
'set userinterfaceonly to True, to protect the user interface, but not macros. If this argument is omitted, protection applies both to macros and to the user interface. 
.Protect Password:="123" , contents:=True, UserInterfaceOnly:=True

 'AutoFilter arrows get visible, with a filter criteria
.Range("A1:B10" ).AutoFilter
.Range("A1:B10" ).AutoFilter Field:=1, Criteria1:= ">35" 
 
End With

End Sub

{% endhighlight %}
 
 
## Using UserInterfaceOnly argument to Allow Grouping:
 
{% highlight vb.net linenos %} 
Sub enableGroupingOnProtectedWs()
'Using UserInterfaceOnly argument to allow grouping on a protected worksheet

With Sheet1
 
'allows grouping on a protected worksheet when user-interface-only protection is turned on. This property is not saved with the worksheet or session.
.EnableOutlining = True

'set userinterfaceonly to True, to protect the user interface, but not macros. If this argument is omitted, protection applies both to macros and to the user interface.
.Protect Password:="123" , contents:=True, UserInterfaceOnly:=True

'to group rows
Rows("12:14").Group

'to ungroup rows
'Rows("12:14").Ungroup

End With

End Sub

{% endhighlight %} 
 
 
 
## Worksheet.Protect Method:
 
Worksheet.Protect Method - Protect Method as it applies to the Worksheet Object:
 
This method Protects a worksheet so that it cannot be modified. If changes are required to be made to a protected worksheet, it is possible to use the Protect method on a protected worksheet if the password is supplied. Also, another method would be to unprotect the worksheet, make the necessary changes, and then protect the worksheet again.
 
Syntax:
`expression.Protect(Password, DrawingObjects, Contents, Scenarios, UserInterfaceOnly, AllowFormattingCells, AllowFormattingColumns, AllowFormattingRows, AllowInsertingColumns, AllowInsertingRows, AllowInsertingHyperlinks, AllowDeletingColumns, AllowDeletingRows, AllowSorting, AllowFiltering, AllowUsingPivotTables)`
 
**expression* Required. An expression that returns a Worksheet object.
 
*Password* till "AllowUsingPivotTables" are all Optional Variants, of which "UserInterfaceOnly" has been discussed in detail above. Set the "UserInterfaceOnly" argument to TRUE (default value is False), to protect the user interface, but not macros. If this argument is omitted, protection applies both to macros and to the user interface. This makes it distinct from the other Optional Variants, some of which have been illustrated below.
 
{% highlight vb.net linenos %}
Sub allowSortingOnProtectedWs()
'Allow sorting option on a protected worksheet (from user interface or with VBA code), without using the UserInterfaceOnly argument.
 
'Sorting can only be performed on unlocked or unprotected cells in a protected worksheet
Sheet1.Unprotect Password:="abc" 
Sheet1.Range("G1:H10").Locked = False
 
'Set the AllowSorting property by using the Protect method arguments. This will allow sorting to be performed on the protected worksheet.
Sheet1.Protect Password:="abc" , contents:=True, AllowSorting:=True

'sort code
Sheet1.Range("G1:H10" ).Sort Key1:=Sheet1.Range("G1" ), Order1:=xlDescending

End Sub
 
 
Sub allowFormattingCellsOnProtectedWs()
'Allow formatting of cells on a protected worksheet (from user interface or with VBA code), without using the UserInterfaceOnly argument.
 
'AllowFormattingCells property is set by using the Protect method arguments. Use of this property allows the user to change all formats, but not to unlock or unhide ranges.
Sheet1.Protect Password:="123" , AllowFormattingCells:=True

'code to format cells
Sheet1.Range("G1:H10" ).Font.Bold = True
Sheet1.Range("A1:B10" ).Font.Color = vbBlue

End Sub
 
 
Sub allowFormattingColumnsRowsOnProtectedWs()
'Allow formatting of columns/rows (ie. whether or not columns/rows can resized) on a protected worksheet (from user interface or with VBA code), without using the UserInterfaceOnly argument.
 
'AllowFormattingColumns & AllowFormattingRows property is set by using the Protect method arguments. Set this property to True to enable columns/rows to be resized when the worksheet is protected. The default value is False.
Sheet1.Protect Password:="123" , AllowFormattingColumns:=True, AllowFormattingRows:=True

'code to format columns and rows
Sheet1.Columns(2).ColumnWidth = 25
Sheet1.Rows("4:5" ).RowHeight = 25

End Sub

{% endhighlight %}

# Reference
1. [Run VBA Macros on Protected Worksheet](http://www.globaliconnect.com/excel/index.php?option=com_content&view=article&id=117:run-vba-macros-on-protected-worksheet-unprotect-a-protect-userinterfaceonly-argument-worksheetprotect-method&catid=79&Itemid=475)
2. [How do I avoid run-time error when a worksheet is protected in MS-Excel?](http://stackoverflow.com/questions/445519/how-do-i-avoid-run-time-error-when-a-worksheet-is-protected-in-ms-excel)
3. [How to protect cells in Excel but allow these to be modified by VBA script](http://stackoverflow.com/questions/125449/how-to-protect-cells-in-excel-but-allow-these-to-be-modified-by-vba-script)