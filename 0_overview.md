# Overview

## Terminology

| Term | Definition
| --- | ---
| IDE | Integrated Development Environment
| VS | Microsoft Visual Studio
| R# | JetBrains ReSharper
| AJAX | Asynchronous JavaScript and XML
| API | Application Programming Interface
| DRY | Don’t Repeat Yourself
| YAGNI | You Ain't Gonna Need It

## History

This document started out as a Microsoft Word document and was originally published at CodePlex in 2008 by [Encodo Systems AG](http://encodo.com). It has been updated a few times over the years, but not nearly often enough to keep pace with changes.

This most recent version reflects a conversion to Markdown format and storage in a Git repository to improve version-tracking, collaboration and updates.

It is also a complete overhaul in both content and structure to provide maximum benefit to various readers. It is also now maintained by Marco von Ballmoos (partner and senior developer at [Encodo Systems AG](http://encodo.com)) instead of by Encodo itself.

## Versions

|Version |  Date | Author | Comments
| --- | --- | --- | ---
| 7.0.0 | 29.04.2017 | MvB | Converted the manual from Microsoft Word to Markdown format; rewrote most chapters; removed redundancy; upgraded all styles/patterns from C# 3.5 to C# 7; matched version number to language-version number
| 2.0.0 | Unreleased | MvB | Updated text throughout the “1 − General” and “2 − Design Guide” sections; added “7.17 − Using System.Linq”, “7.29 − Refactoring Names and Signatures” and “7.30 − Loose vs. Tight Coupling” best practices;
| 1.5.3 | Unreleased | MvB | Added reference and notes from “The Little Manual of API Design”; added section “7.15 – Using Partial Classes”.
| 1.5.2 | 19.10.2009 | MvB | Expanded “8.1 – Documentation” with examples; added more tips to the “2.3 – Interfaces vs. Abstract Classes” section; added “7.21 – Restricting Access with Interfaces”; added “5.3.7 – Extension Methods” and “7.18 – Using Extension Methods”.
| 1.5.1 | 24.10.2008 | MvB | Incorporated feedback from the forums at the MSDN Code Gallery.
| 1.5 | 20.05.2008 | MvB | Updated line-breaking section; added more tips for generic methods; added tips for naming delegates and delegate parameters; added rules for object and array initializers; added rules and best practices for return statements; added tips for formatting complex conditions; added section on formatting switch statements.
| 1.4 | 18.04.2008 | MvB | Updated formatting for code examples; added section on using the var keyword; improved section on choosing names; added naming conventions for lambda expressions; added examples for formatting methods; re-organized error handling/exceptions section; updated formatting.
| 1.3 | 31.03.2008 | MvB | Added more tips for documentation; added white-space rules for regions; expanded rules for line-breaking; updated event naming section.
| 1.2 | 07.03.2008 | MvB | Change to empty methods; added conditional compilation section; updated section on comments; made some examples customer-neutral; fixed some syntax-highlighting; reorganized language elements.
| 1.1 | 06.02.2008 | MvB | Updated sections on error handling and naming
| 1.0 | 28.01.2008 | MvB | First Draft
| 0.1 | 03.12.2007 | MvB | Document Created

## Referenced Documents

The style and formatting guidelines draw mostly from in-house programming experience. Most of the following documents were more heavily used in prior versions. References have been included for completeness.

| Date | Title | Authors | Version
| --- | --- | --- | ---
| 13.04.2017 | [Patterns and Practices in C# 7](https://www.infoq.com/articles/Patterns-Practices-CSharp-7) | Jonathan Allen |
| 20.07.2015 | [Microsoft C# Coding Conventions](https://msdn.microsoft.com/en-us/library/ff926074.aspx) | Microsoft | VS2015
| 01.07.2011 | [IDesign C# Coding Standards](https://www.scribd.com/document/236016479/IDesign-C-Coding-Standard-2-4) | Juval Lowy | 2.4
| 19.05.2011 | [Optional argument corner cases, part four](http://blogs.msdn.com/b/ericlippert/archive/2011/05/19/optional-argument-corner-cases-part-four.aspx) | Eric Lippert |
| 01.11.2008 | [Microsoft Framework Design Guidelines](https://msdn.microsoft.com/en-us/library/ms229042(v=vs.110).aspx) | Krzysztof Cwalina and Brad Abrams | 2.0
| 19.06.2008 | [The Little Manual of API Design](http://www4.in.tum.de/~blanchet/api-design.pdf) (PDF) | Jasmin Blanchette |
| 19.05.2005 | [Coding Standard: C#](http://www.sourceformat.com/pdf/cs-coding-standard-philips.pdf) (PDF) | Philips Medical Systems | 1.3
| 26.01.2005 | [Microsoft Internal Coding Guidelines](https://blogs.msdn.microsoft.com/brada/2005/01/26/internal-coding-guidelines/) | Brad Abrams | 2.0