= Why a Sequence Diagram Generator? =

[[File:DiagramSequenceGenerator-SimpleExample.png|200px|right|Simple Diagram]]

* To be able to inspect the calls made during the execution of the program without opening a debugger.
* Understand a piece of software, a package, some interactions between the packages
* Detect some flaws
* Make schemes for your slides ;)

This produce the kind of diagram on the right.

= How to Use it? =

== Load it ==

'''First, save your image! One never knows what could happen...'''

In the Lam Store load the latest Bundle named: <code>SequenceDiagramCore</code> with the version <code>Released</code>.
Some compatibility packages will be loaded too.

'''NB: the loading and execution has not been tested with a SW2300 version < SP13 - 444'''

== Generate your PlantUML code ==

There are 2 ways to generate the diagram: the first is by using the API (reliable, but not so user friendly), the second is by using the probes (less reliable, but much more user friendly).

=== API call version ===

The available API is on the '''class side''' of <code>SequenceDiagram.SequenceDiagram</code> in the <code>initialize-release</code> protocol.

The main method is: <syntaxhighlight lang="smalltalk" inline>#run:on:fileIn:</syntaxhighlight>
Arguments:
# The block of code that is watched during its execution (can use some variable outside of the block)
# The list of Smalltalk packages where the execution of the block is recorded. Only the classes and methods of these packages are going to appear on the generated diagram. '''It is strongly advised to not put any kernel package in this list!'''
# The file where the plantUML code is written

You can write in a stream instead of a file by replacing 'fileIn:' by 'writeIn:'.

Example:
<syntaxhighlight lang="smalltalk">
SequenceDiagram.SequenceDiagram
    run: [Point x: 10 y: 10] 
    on: #('Graphics-Geometry')  
    fileIn: 'C:\temp\point.txt'.
</syntaxhighlight >

=== Probes version ===

''The current implementation of this version is based on AST modification and recompilation with additions of probes. The probed method could be not executable or present some defects. However, it is guarantied that removing the probes will restore the original method.''

If you consent to use it though:
#Open a Browser
#Select the piece of code you want to add your probe in (the selection may not be exact, it will work, but some strange results can happen with cascades). If you use a method currently used by the system, you may result in a crash.
#Open the contextual menu (middle click by default)
#Click on '''Generate Sequence Diagram'''
#In the new window, you can select the packages you want to instrument and the name of the file (and the [[transformers]] too)
#Click on Ok when done: the method will be recompiled and probed. '''Do not publish your code source nor edit and save the method you probed, or you gonna have to restore the original version yourself!'''
#Execute any piece of code you want that call the probed method
#Remove the probe (you may want to use 'Remove all probes')

==== Next features? / Limitations ====

* Probes cannot be edited once created, you need to destroy and create again
* The code of the probed method becomes dirty but without modification
* You need to remove the two probes (the start probe and the end probe)
* The modification of the AST is not accurate, so it may result in more instrumented code that you wanted: the code instrumented is between the two probes.

== Create your Diagrams ==

'''Do not upload the generated file on a public server to generate the picture! You may share some data that are confidential'''

So, you need to use a local generator:
# Download http://plantuml.com/download
# Execute this command line:
<syntaxhighlight lang="bash">
java -DPLANTUML_LIMIT_SIZE=18192 -Xmx4096m -jar  .\plantuml.jar -tsvg -gui
</syntaxhighlight>
# Set up the directory where your source code diagram are

== How to read the diagram ==

* Each vertical line represent an instance of a class
* Horizontal lines are the method calls:
** Filled line represent the start of the method call
** Dotted line represent the end of the method call

Example:

[[File:DiagramSequenceGenerator-SimpleExample.png]]

The <code>#x:y</code> message is invoked on the metaclass instance of Point. This method invokes <code>#setX:setY:</code> on the newly created Point instance, then returns, and the <code>#x:y</code> message return which ends the invocation.


=== Multithreading ===

Each color for the links between the instances represent the process that executed the call. Several colors mean that several processes are involved (colors are randomly choosen).

= Add Fancy Transformers to Enhance your Masterpiece = 

Maybe you diagram is too huge to be generated or you want to get rid of some noise, you can use these transformers:
* <code>SequenceDiagramModelTransformerFilter</code>: remove some methods by name
* <code>SequenceDiagramModelTransformerResetInstanceNumbers</code>: rename properly the ids of the instances
* <code>SequenceDiagramModelTransformerUniqueInstance</code>: group all the instances of a class in the diagram
* <code>SequenceDiagramModelTransformerLoopCondenser</code>: avoid the repetitions of the calls that are inside loops by grouping them. It is strongly advised to use the transformer <code>SequenceDiagramModelTransformerUniqueInstance</code> before.

The order of the transformer is important, the first selected is the first that will be applied.

Example:
<syntaxhighlight lang="smalltalk">
SequenceDiagram.SequenceDiagram
    run: [Point x: 10 y: 10] 
    on: #('Graphics-Geometry')  
    fileIn: 'C:\temp\point.txt'
    withTransformers: 
         (OrderedCollection with: SequenceDiagramModelTransformerUniqueInstance new
                            with: SequenceDiagramModelTransformerResetInstanceNumbers new).
</syntaxhighlight >
Result:
[[File:SequenceDiagramGenerator-TransformedExample.png]]

Full example:
<syntaxhighlight lang="smalltalk">
SequenceDiagram.SequenceDiagram
    run: [Point x: 10 y: 10] 
    on: #('Graphics-Geometry')  
    fileIn: 'C:\temp\point.txt'
    withTransformers: 
         (OrderedCollection with: SequenceDiagram.SequenceDiagramModelTransformerUniqueInstance new
                            with: SequenceDiagram.SequenceDiagramModelTransformerResetInstanceNumbers new
                            with: SequenceDiagram.SequenceDiagramModelTransformerLoopCondenser new).
</syntaxhighlight >


Feel free to create new transformers if you need others. But keep in mind that you should only modify the generated model and not  the generator.

= Overall Limitations =

On a general way, this tool modify the intrinsic core of the methods. Some defects can happen in the on-the-edge cases. 

About the result, the various ST processes that calls the instrumented methods are identified as one unique process. The diagram will then show all the processes without grouping the method calls by thread.

= Reports Bugs or Missing features =

Go to the [http://phab.fremont.lamrc.net/project/profile/186/ Phabricator Project]
