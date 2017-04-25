# Managing Change

## Modifying Interfaces

* Avoid introducing breaking changes. Wherever possible, retain old overloads/names for one extra major version.
* Document breaking changes in the release notes, including upgrade instructions.

## Marking Members as Obsolete

Include the version number in the message. For example:

```csharp
[Obsolete(""Since 4.0: Use IMetaElementSearchAspect instead.")]
```

## Refactoring Names and Signatures

The whole point of having an agile process and lots of automated tests is to be able to quickly improve designs and accommodate new functionality. Very often, this involves quite aggressive refactoring. Aggressive refactoring means that code that compiled with a previous version of framework may no longer compile with the latest version.
In each case where such a change is to be made, the following points must be considered:

* Will customer code be affected? A “customer” in this case is _anyone_ who consumes your code: both internal and external developers.
* Do you have access to all uses of the code in order to be able to update it?
* Is all of the code that depends on code integrated at least daily in order to catch any usages you might have forgotten?
* Is the area you are changing covered by automated tests?
* Just how important is the change?
* Can you make the change in a non-destructive manner by introducing an optional parameter or an overload instead?
* Are you willing to introduce a compile error into customer code?

With an agile methodology, the answer to the question “should you?” is quite often “yes”. Cleaner, tighter and more logical code is more maintainable and self-explanatory code.

## Roadmap for Safe Obsolescence

If the change is localized, you can of course make it right away. If not, you may need to go the long route described below:

1.	Mark the old version of the feature as obsolete.
2.	Create the new feature with a different name so that it does not collide with the existing feature.
3.	Rewrite the code so that the main code path uses the new feature but also incorporates the old version as well.
4.	Add an issue to complete the next stage of refactoring in the next release.
5.	Make and distribute a point release.
6.	In the next release, remove the obsolete feature and all handling for it.
7.	Rename the new feature to the desired name but retain a copy with the temporary name as well, marking it as obsolete.
8.	Update the issue to indicate that it should be completed in the next release.
9.	Make and distribute a point release.
10.	In the next release, remove the obsolete version with the temporary name and the refactoring is complete.
11.	Close the issue.

Because the steps outlined above require the patience of a saint, they should really only be used for features that absolutely _must_ be refactored but that absolutely _cannot_ break customer code. In all other cases, refactor away and make sure that repair instructions for the compile error are included in the release notes.