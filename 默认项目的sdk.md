```xml
<Project Sdk="Godot.NET.Sdk/4.4.1">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <EnableDynamicLoading>true</EnableDynamicLoading>
    <RootNamespace>GodotSwf测试项目</RootNamespace>
	  <EnableDynamicLoading>true</EnableDynamicLoading>
	  <PublishAot>true</PublishAot>
  </PropertyGroup>
  <ItemGroup>
	<TrimmerRootAssembly Include="GodotSharp" />
	<TrimmerRootAssembly Include="Clipper2Lib" />
	<TrimmerRootAssembly Include="Godot SWF" />
	<TrimmerRootAssembly Include="LibTessDotNet" />
	<TrimmerRootAssembly Include="LibTessDotNet.Double" />
	<TrimmerRootAssembly Include="SharpCompress" />
	<TrimmerRootAssembly Include="SwfLib" />
	<TrimmerRootAssembly Include="$(TargetName)" />
	
    <Reference Include="Clipper2Lib">
      <HintPath>Clipper2Lib.dll</HintPath>
    </Reference>
    <Reference Include="Godot SWF">
      <HintPath>Godot SWF.dll</HintPath>
    </Reference>
    <Reference Include="LibTessDotNet">
      <HintPath>LibTessDotNet.dll</HintPath>
    </Reference>
    <Reference Include="LibTessDotNet.Double">
      <HintPath>LibTessDotNet.Double.dll</HintPath>
    </Reference>
    <Reference Include="SharpCompress">
      <HintPath>SharpCompress.dll</HintPath>
    </Reference>
    <Reference Include="SwfLib">
      <HintPath>SwfLib.dll</HintPath>
    </Reference>
  </ItemGroup>
</Project>
```