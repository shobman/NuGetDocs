#Creating a Package

There are a few approaches to creating a package. Most packages are very simple and contain 
a single assembly. In those cases, there are some very easy ways to create packages. 
Then there are the complex packages, which might require a little more work. 

<div class="block-callout-warning">
	<strong>Pre-requisites</strong><br>
    <a href="https://dist.nuget.org/win-x86-commandline/latest/nuget.exe">Download NuGet.exe</a> and save it to a path of your choice.<br>
	Add this path to the <a href="https://msdn.microsoft.com/en-us/library/office/ee537574(v=office.14).aspx">PATH Environment Variable</a>.
</div>

<div class="block-callout-info">
	If you are <strong>GUI inclined</strong>, use the <a href="/Create/using-a-gui-to-build-packages" class="alert-link">Package Explorer GUI to create packages</a>.<br>
	<small>Note: This tool is OSS, not supported by the NuGet team, and is not signed.</small></p>
</div>
##Assembly
If you have an assembly, you can easily generate a <span class="text-primary">.nuspec</span> file from metadata within the assembly 
and create a package.

<div class="block-callout-info">
	<a href="http://docs.nuget.org/Create/Nuspec-Reference">.nuspec</a> file is a manifest that uses XML to describe a package. The manifest is used to build a package and is also stored in the package after the package is built.
</div>

###Create a .nuspec file
A .nuspec file uses its XML-based manifest to build a package. The .nuspec file is also stored in the package after the package is built. 
To create a .nuspec file, run the following command: 

    nuget spec MyAssembly.dll
	
The .nuspec file should look similar to the following:

	<?xml version="1.0"?>
		<package xmlns="http://schemas.microsoft.com/packaging/2016/06/nuspec.xsd">
			<metadata>
				<id>MyPackageId</id>
				<version>1.0</version>
				<authors>philha</authors>
				<owners>philha</owners>
				<licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
				<projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
				<iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
				<requireLicenseAcceptance>false</requireLicenseAcceptance>
				<description>Package description</description>
				<tags>Tag1 Tag2</tags>
				<dependencies>
				  <dependency id="SampleDependency" version="1.0" />
				</dependencies>
			</metadata>
		</package>


###Pack
To create your package, run the following command:

    nuget pack MyAssembly.nuspec
	
##Project
Creating a package from a .csproj file or a .vbproj file is convenient because other packages that have been installed into your project will be referenced as dependencies.

###Create a .nuspec file
    nuget spec
This creates a .nuspec file that includes tokens meant to be replaced at pack time, based on the project metadata.
NuGet will replace the tokens only when you execute the pack command with the project file, and not with the .nuspec file. 
You can then edit this .nuspec file if you want to customize it. For example, if you don't want a token replacement for some fields, you can hard-code them in the .nuspec instead.
The following is a list of the supported replacement tokens.
<table class="reference">
    <tr><th>Token</th><th>Source</th></tr>
    <tr>
        <td>$id$</td>
        <td>The Assembly name</td>
    </tr>
    <tr>
        <td>$version$</td>
        <td>The assembly version as specified in the assembly&#8217;s <code>AssemblyVersionAttribute</code> (or <code>AssemblyInformationalVersionAttribute</code> if present).</td>
    </tr>
    <tr>
        <td>$author$</td>
        <td>The company as specified in the <code>AssemblyCompanyAttribute</code>.</td>
    </tr>
        <tr>
        <td>$description$</td>
        <td>The description as specified in the <code>AssemblyDescriptionAttribute</code>.</td>
    </tr>
</table>

###Pack
    nuget pack MyProject.csproj
<div class="block-callout-warning">
	You must run nuget pack on the project file, not the .nuspec file itself. But the .nuspec file will in fact get picked up.<br>
	By default, the nuget pack command excludes any folder that starts with a "x", such as .git or .hg.<br>
	When you directly target a .nuspec file, the pack command will NOT replace any tokens in the .nuspec file.<br>
</div>

###Files
If you want to explicitly define which files are included in the package, you can use a node in the .nuspec file.
For example, suppose you want to add all the files from some arbitrary other folder into the package, you'd add:

    <files>
		<file src="..\..\SomeRoot\**\*.*" target="" /> 
	</files>

<div class="block-callout-info">
	Note that only the files defined inside the node will be included in the package. If you include an empty node in the .nuspec file, no content files will be included; only the lib folder will. 
</div>

###Referenced projects
If the project references other projects, you can add the referenced projects as part of the package, or as dependencies by using the -IncludeReferencedProjects option, as follows:

    nuget pack A.csproj -IncludeReferencedProjects
	

<div class="block-callout-info">
	Note: If a referenced project includes .nuspec file that has the same name, then NuGet adds that referenced project as a dependency instead. 
</div>

###Build configuration
By default, NuGet will use the default build configuration set in the project file (typically Debug). To pack files from a different build configuration (e.g., Release), run the following command:

    nuget pack MyProject.csproj -Prop Configuration=Release
	
###Symbols
You can also use the -Symbols flag to include a symbols package as well. A symbols package allows others to step into your package's code in the debugger.

    nuget pack MyProject.csproj -Symbols
	

##Convention-based working directory
Some packages contain more than just assemblies. They may contain the following:<br>
Content and source code that should be injected into the target project.<br>
<a href"https://docs.nuget.org/Create/Configuration-File-and-Source-Code-Transformations">Configuration file and source code transformations.</a>


Some packages might have scripts, like init.ps, for example. Legacy packages that are installed in the packages.config world might use install.ps1 scripts to setup the project. 
<div class="block-callout-warning">
	Important: This is no longer supported in project.json.
</div>
To create a convention-based package, you can lay out a directory structure that uses the following NuGet conventions:<br>
__tools__ - The tools folder of a package is for PowerShell scripts and programs accessible from the Package Manager Console. After the folder is copied to the target project, it is added to the `$env:Path (PATH) environment variable. <br>
__lib__ - Assemblies (.dll files) in the lib folder are added as assembly references when the package is installed.<br>
__content__ - Files in the content folder are copied to the root of your application when the package is installed. <br>
__build__ - The build folder of a package is for MSBuild targets files that are automatically inserted into the .csproj file of the application.<br>


__Think of the Content folder as the root of your target application.__ 
For example, if you want a package to add an image in the /images directory of the target application, make sure to place the image in the Content/images folder of the package.

###Create the .nuspec file
To create a spec file from scratch, run the following command:

    nuget spec

This will create an XML file with the .nuspec file extension, as follows:

    <?xml version="1.0"?>
    <package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
	    <metadata>
	    <id>MyPackageId</id>
	    <version>1.0</version>
	    <authors>philha</authors>
	    <owners>philha</owners>
	    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
	    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
	    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
	    <requireLicenseAcceptance>false</requireLicenseAcceptance>
	    <description>Package description</description>
	    <tags>Tag1 Tag2</tags>
	    <dependencies>
		    <dependency id="SampleDependency" version="1.0" />
	    </dependencies>
	    </metadata>
    </package>

Edit this file to fill in the values appropriate for your package. Then, create the folders needed by your package and copy the correct content in each folder. 

###Pack
Navigate to the root of the directory, and run the following command:

    nuget pack YourPackage.nuspec


##Next Steps
Congratulations! You have created your first <span class="text-primary">nupkg</span>. You can move on to the <a href="#">next topic</a> where you will learn more about publishing your nuget package.

##Related Reading
<a href="#">Build script to assemble your package</a><br>
<a href="#">Best practices from the community</a><br>
<a href="#">Adding further functionalities to your package</a>