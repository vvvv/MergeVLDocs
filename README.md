# MergeVLDocs
Merging VL documents with traditional tools is hard. 
This tool is the first step towards an easy-to-use visual merge tool.
  
### VL Documents
Merging patches shouldn't be harder, but easier to merge than textual code.
Why?  
A document is a tree of elements, where each element is identifiable via its Id (a GUID). So relating documents in different versions should be mostly about relating elements with the same Id.

### Problem space
Merging is about taking the difference 
 * from *Local* to *Base* 
 * from *Remote* to *Base*
 * combine these in a certain way

Let's distinguish different cases:
 * **directly conflicting changes**: E.g. 
   * a node was moved on *Local* and on *Remote*
   * The version of a dependency changed on *Local* and *Remote* ...
 * **structural changes**: E.g
   * Dev A moved a bunch of definitions into a new *Category* or *Group*. Dev B changed one of those definitions. 
 * Changes were made in the **same patch or the same canvas. But on different ends**
 * Changes were made on **different patches/canvases**
 
Wishful thinking:
 * **directly conflicting changes**: nah. can't merge. But maybe we can come up with something that doesn't feel so harsh. These changes sound like real bad conflicts, but then again: often your changes are not really about nodes that got moved...
 * **structural changes**: This is unmergeable for now since the only way to do those structural changes (via patch editor) is to copy and paste the definitions resulting in new GUID-Ids for all the elements. 
   *Note: it would be so cool to solve that in the future*
 * **same patch/canvas, different ends**: This should be mergeable. But it's not likely that the different ideas will merge semantically and the result will just run.
 * **different patches/canvases**: Sounds easy to be merged. But ideas might still collide: Dev A changed an interface and all its implementations. Dev B added an implementation of the old interface as known from *Base*

### First approach
Let's build a small application that gets called by git merge. Let's access the different versions (*Base*, *Local*, *Remote*) in the same way that p4merge and other merge tools do: via command-line arguments.
In this first sketch, we load the three VL documents just as an XElement structure. 

In this first naive implementation, we try to come up with a solution that covers most cases...
So let's just work on an XElement and XAttribute level.

* Align XElements via their Id 
* Merge their Attributes
* If one element got new child elements or attributes: take them
* If the other one also added new child elements: take them too! *these should come with different GUIDs*
* If some child elements or attributes got removed, then remove them (no matter if they got changed in the other version). Intuition for now: focus on the bold changes
* If a certain attribute got changed in both versions: Choose remote. That's a kind move. Let the other dev win. In rare cases choose the *Base* value (e.g. "LanguageVersion" of the document). 

There are many things to discuss and to test. But it's a start.
It's obvious that this approach can mess up the patches as much as a line-based merge tool. But at least certain kinds of errors can't occur. 
 * It should always be a valid document (even though with maybe many compile-time errors), but the integrity of each element should be preserved. Which is not guaranteed with line-based merges.
 * We can tackle different elements in different ways. For now 
   * *LanguageVersion*-Attribute in the Document
   * *NodeReference*-Element

### Goals
After merging: 
 * show which parts were hard to merge and make it easy to navigate to those patches and inspect what happened.
 * make it easy to do some fixes if the auto-magic merge failed.
 * Have an installer that takes care of everything or even make it part of the official vvvv installer.

### Setup
For now, it is manual.

* Download the PreRelease 
* Unzip, make sure the name containing the exe is called just `MergeVLDocs` and move that folder under `C:\Program Files` or whereever you want it
* Go to your windows user folder (typically `C:\Users\__MEMEMEME__`) and edit the .gitconfig file. Consider making a copy of the original file before changing it.
```
  [merge]
    tool = MergeVLDocs
```
```
  [mergetool "MergeVLDocs"]
    cmd = \"C:/Program Files/MergeVLDocs/MergeVLDocs.exe\" \"$BASE\" \"$LOCAL\" \"$REMOTE\" \"$MERGED\"
    path = C:/Program Files/MergeVLDocs/MergeVLDocs.exe
```
make sure the path above is correct and the executable is at the correct location.
Note that the path is made of slashes, not back-slashes. (Sorry for the inconvenience)

This configures our merge tool to be called regardless of the file type. The idea here is that our tool should do its job for VL-files, but call the previously configured tool in all other cases.
* For now: Install p4merge. This tool gets called by MergeVLDocs if it detects that it is not responsible for the file type in question. 

### Merge Tests
You can test the tool with your library or project VL docs, but that's not recommended for now.
https://github.com/gregsn/MergeVLDocsExamples is here to exemplify merge problems and is here to test our tool.
