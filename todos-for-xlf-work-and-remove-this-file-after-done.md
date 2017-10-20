# This branch is made for adopting xlf tasks to Roslyn repository, please remove this file when it is done and merge it back to upstream.

# End goal
When developer runs build from Visual Studio or build script, the xlf files will be updated/generated, and satellite assemblies will be generated. And in official/CI build, the build will fail if XLF is out of date.

Satellite assemblies should also be included in VSIX file and insert to Visual Studio.

# Blocker
## alternative of ALINK
[Clause](https://connect.microsoft.com/VisualStudio/feedback/details/611727/localized-build-with-free-form-assemblyinformationalversion-causes-alink-warning-al1053)

We need to wait for the alternative of ALINK. So [this](https://github.com/wli3/roslyn/commit/cc465bfb5a9e3e3d72a7f71c4072c6855f30fa5a) can be reverted before check-in.

## Extra line on generated AssemblyInfoFile _$(TargetName).resources.cs_
If build using core MSBuild(build on Linux), there will be an error as following

```
/home/wul/roslyn/Binaries/Obj/BasicCodeAnalysis/Debug/cs/Microsoft.CodeAnalysis.VisualBasic.resources.cs(1,1): error CS1010: Newline in constant
```
It is due to [MSBuild bug](https://github.com/Microsoft/msbuild/issues/937). On .NET Core, msbuild is not escaping the code with WriteCodeFragment. Workaround (for now) would be to adjust generated assembly info to not have that newline.

# TODO
## Fix when two VSCT nodes has the same Id attribute
This [pull request](https://github.com/dotnet/xliff-tasks/pull/33
)'s tests are valid. However solution is not good, it will cause all current XLF to update with new ID that has VSCT GUID+ID.

The solution might be change Roslyn itself to avoid duplicated VSCT ID in [here](https://github.com/dotnet/roslyn/blob/614299ff83da9959fa07131c6d0ffbc58873b6ae/src/VisualStudio/Core/Def/Commands.vsct#L284) and [here](https://github.com/dotnet/roslyn/blob/614299ff83da9959fa07131c6d0ffbc58873b6ae/src/VisualStudio/Core/Def/Commands.vsct#L294)

Or update VSCT ID algorithm that will not cause massive update of generated XLF file.

## Check in generated XLF file

## Include generated resource files into VSIX

Satellite assemblies should also be included in VSIX file and insert to Visual Studio. However, this will conflict with existing localization pipeline, we need to do a proper hand over of the old and new system.


# What is xliff-tasks
https://github.com/dotnet/xliff-tasks is for automatically generate xlf file and satellite assemblies in order to localize product.

After adding xlf tasks as package reference. By using `/p:UpdateXlfOnBuild=true`, it will generate/update xlf files according to resource files. These xlf files should be checked in and localization team will add their translation to the xlf files.

Also during the build process, xlf tasks will use these xlf files to generate satellite assemblies.