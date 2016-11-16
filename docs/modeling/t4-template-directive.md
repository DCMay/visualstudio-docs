---
title: "T4 Template Directive | Microsoft Docs"
ms.custom: ""
ms.date: "11/04/2016"
ms.reviewer: ""
ms.suite: ""
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 2b0a8e04-6fee-4c6c-b086-e49fc728a3ed
caps.latest.revision: 10
author: "alancameronwills"
ms.author: "awills"
manager: "douge"
translation.priority.ht: 
  - "cs-cz"
  - "de-de"
  - "es-es"
  - "fr-fr"
  - "it-it"
  - "ja-jp"
  - "ko-kr"
  - "pl-pl"
  - "pt-br"
  - "ru-ru"
  - "tr-tr"
  - "zh-cn"
  - "zh-tw"
---
# T4 Template Directive
A [!INCLUDE[vsprvs](../code-quality/includes/vsprvs_md.md)] T4 text template usually starts with a `template` directive, which specifies how the template should be processed. There should be no more than one template directive in a text template and any files that it includes.  
  
 For a general overview of writing text templates, see [Writing a T4 Text Template](../modeling/writing-a-t4-text-template.md).  
  
## Using the Template Directive  
  
```  
<#@ template [language="VB"] [compilerOptions="options"] [culture="code"] [debug="true"] [hostspecific="true"] [inherits="templateBaseClass"] [visibility="internal"] [linePragmas="false"] #>  
```  
  
 The `template` directive has several attributes that allow you to specify different aspects of the transformation. All the attributes are optional.  
  
## compilerOptions attribute  
 Example:  
 `compilerOptions="optimize+"`  
  
 Valid values:  
 Any valid compiler options. For more information, see [C# Compiler Options Listed by Category](/dotnet/csharp/language-reference/compiler-options/listed-by-category) and [Visual Basic Compiler Options Listed by Category](/dotnet/visual-basic/reference/command-line-compiler/compiler-options-listed-by-category).  
  
 Ignored for run-time (preprocessed) templates.  
  
 These options are applied when the template has been converted into [!INCLUDE[csprcs](../data-tools/includes/csprcs_md.md)] or [!INCLUDE[vb_current_short](../debugger/includes/vb_current_short_md.md)], and the resulting code is compiled.  
  
## culture attribute  
 Example:  
 `culture="de-CH"`  
  
 Valid values:  
 "", the invariant culture, which is the default.  
  
 A culture expressed as a string in the form xx-XX. For example, en-US, ja-JP, de-CH, de-DE. For more information, see <xref:System.Globalization.CultureInfo?displayProperty=fullName>.  
  
 The culture attribute specifies the culture to use when an expression block is converted to text.  
  
## debug attribute  
 Example:  
 ```  
debug="true"  
```  
  
 Valid values:  
 `true, false`. False is the default.  
  
 If the `debug` attribute is `true`, the intermediate code file will contain information that enables the debugger to identify more accurately the position in your template where a break or exception occurred.  
  
 For design-time templates the intermediate code file will be written to your **%TEMP%** directory.  
  
 To run a design-time template in the debugger, save the text template, then open the shortcut menu of the text template in Solution Explorer, and choose **Debug T4 Template**.  
  
## hostspecific attribute  
 Example:  
 ```  
hostspecific="true"  
```  
  
 Valid values:  
 `true, false, trueFromBase`. False is the default.  
  
 If you set the value of this attribute to `true`, a property named `Host` is added to the class generated by your text template. The property is a reference to the host of the transformation engine, and is declared as <xref:Microsoft.VisualStudio.TextTemplating.ITextTemplatingEngineHost>. If you have defined a custom host, you can cast it to the custom host type.  
  
 Because the type of this property depends on the type of host, it is only useful if you are writing a text template that works only with a specific host. It’s applicable to [design-time templates](../modeling/design-time-code-generation-by-using-t4-text-templates.md), but not [run-time templates](../modeling/run-time-text-generation-with-t4-text-templates.md).  
  
 When `hostspecific` is `true` and you are using [!INCLUDE[vsprvs](../code-quality/includes/vsprvs_md.md)], you can cast `this.Host` to IServiceProvider to access [!INCLUDE[vsprvs](../code-quality/includes/vsprvs_md.md)] features. You can also use `Host.ResolvePath(filename)` to obtain the absolute path of a file in the project. For example:  
  
```c#  
<#@ template debug="false" hostspecific="true" language="C#" #>  
<#@ output extension=".txt" #>  
<#@ assembly name="EnvDTE" #>  
<#@ import namespace="EnvDTE" #>  
<#@ import namespace="System.IO" #>  
<# // Get the Visual Studio API as a service:  
 DTE dte = ((IServiceProvider)this.Host).GetCOMService(typeof(DTE)) as DTE;    
#>  
Number of projects in this solution: <#=  dte.Solution.Projects.Count #>  
  
<#  
 // Find a path within the current project:  
 string myFile = File.ReadAllText(this.Host.ResolvePath("MyFile.txt"));  
#>  
Content of myFile is:  
<#= myFile #>  
  
```  
  
 If you use the `inherits` and `hostspecific` attributes together, specify host="trueFromBase" in the derived class and host="true" in the base class. This avoids a double definition of the `Host` property in the generated code.  
  
## language attribute  
 Example:  
 `language="VB"`  
  
 Valid values:  
 `C#` (default)  
  
 `VB`  
  
 The language attribute specifies the language ([!INCLUDE[vbprvb](../code-quality/includes/vbprvb_md.md)] or [!INCLUDE[csprcs](../data-tools/includes/csprcs_md.md)]) to use for the source code in statement and expression blocks. The intermediate code file from which the output is generated will use this language. This language is not related to the language that your template generates, which can be any kind of text.  
  
 For example:  
  
```vb#  
<#@ template language="VB" #>  
<#@ output extension=".txt" #>  
Squares of numbers:  
<#  
  Dim number As Integer  
  For number = 1 To 4  
#>  
  Square of <#= number #> is <#= number * number #>  
<#  
  Next number  
#>  
  
```  
  
## inherits attribute  
 You can specify that the program code of your template can inherit from another class, which can also be generated from a text template.  
  
### Inheritance in a run-time (preprocessed) text template  
 You can use inheritance between run-time text templates to create a basic template that has several derived variants. Run-time templates are those that have the **Custom Tool** property set to **TextTemplatingFilePreprocessor**. A run-time template generates code that you can call in your application to create the text defined in the template. For more information, see [Run-Time Text Generation with T4 Text Templates](../modeling/run-time-text-generation-with-t4-text-templates.md).  
  
 If you do not specify an `inherits` attribute, a base class and a derived class are generated from your text template. When you specify an `inherits` attribute, only the derived class is generated. You can write a base class by hand, but it must provide the methods that are used by the derived class.  
  
 More typically, you specify another preprocessed template as the base class. The base template provides common blocks of text, which can be interleaved with text from the derived templates. You can use class feature blocks `<#+ ... #>` to define methods that contain text fragments. For example, you can place the framework of the output text in the base template, providing virtual methods that can be overridden in derived templates:  
  
 Run-time (preprocessed) text template BaseTemplate.tt:  
 ```scr  
This is the common header.  
<#   
  SpecificFragment1();   
#>  
A common central text.  
<#   
  SpecificFragment2();   
#>  
This is the common footer.  
<#+   
  // Declare abstract methods  
  protected virtual void SpecificFragment1() { }  
  protected virtual void SpecificFragment2() { }  
#>  
  
```  
  
 Run-time (preprocessed) text template DerivedTemplate1.tt:  
 ```c#  
<#@ template language="C#" inherits="BaseTemplate" #>  
<#   
  // Run the base template:  
  base.TransformText();  
#>  
<#+  
// Provide fragments specific to this derived template:  
protected override void SpecificFragment1()  
{  
#>  
   Fragment 1 for DerivedTemplate1  
<#+  
}  
protected override void SpecificFragment2()  
{  
#>  
   Fragment 2 for DerivedTemplate1  
<#+  
}  
#>  
  
```  
  
 Application code to invoke DerivedTemplate1:  
 ```c#  
Console.WriteLine(new DerivedTemplate().TransformText());  
```  
  
 Resulting output:  
 ```  
This is the common header.  
   Fragment 1 for DerivedTemplate1  
A common central text.  
   Fragment 2 for DerivedTemplate1  
This is the common footer.  
```  
  
 You can build the base and derived classes in different projects. Remember to add the base project or assembly to the derived project’s references.  
  
 You can also use an ordinary hand-written class as the base class. The base class must provide the methods used by the derived class.  
  
> [!WARNING]
>  If you use the `inherits` and `hostspecific` attributes together, specify hostspecific="trueFromBase" in the derived class and host="true" in the base class. This avoids a double definition of the `Host` property in the generated code.  
  
### Inheritance in a design-time text template  
 A design-time text template is a file for which **Custom Tool** is set to **TextTemplatingFileGenerator**. The template generates an output file of code or text, which forms part of your [!INCLUDE[vsprvs](../code-quality/includes/vsprvs_md.md)] project. To generate the output file, the template is first translated into an intermediate program code file, which you do not usually see. The `inherits` attribute specifies the base class for this intermediate code.  
  
 For a design-time text template, you can specify any base class that is derived from <xref:Microsoft.VisualStudio.TextTemplating.TextTransformation?displayProperty=fullName>. Use the `<#@assembly#>` directive to load the assembly or project that contains the base class.  
  
 For more information, see ["Inheritance in Text Templates" in Gareth Jones’ Blog](http://go.microsoft.com/fwlink/?LinkId=208373).  
  
## LinePragmas attribute  
 Example:  
 `linePragmas="false"`  
  
 Valid values:  
 `true` (default)  
  
 `false`  
  
 Setting this attribute to false removes the tags that identify your line numbers within the generated code. This means that the compiler will report any errors by using line numbers of the generated code.This gives you more debugging options, as you can choose to debug either the text template or the generated code.  
  
 This attribute can also help if you’re finding the absolute filenames in pragmas are causing distracting merges under source code control.  
  
## Visibility attribute  
 Example:  
 `visibility="internal"`  
  
 Valid values:  
 `public` (default)  
  
 `internal`  
  
 In a runtime text template, this sets the visibility attribute of the generated class. By default, the class is part of the public API of your code, but by setting `visibility="internal"` you can make sure that only your code can use the text-generating class.