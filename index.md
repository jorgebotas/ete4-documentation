
# Tree Master Class

There is an important update to ETE's TreeNode class: it is now written in [Cython](https://cython.org) for higher efficiency.
However, there is one caveat, TreeNode's no longer have features, they have properties: `TreeNode.props`.
This `props` attribute is a dictionary containing node related properties,
mimicking the old functionality of features, but encapsulated in a dictionary. 
Thus, there are a couple of common ETE commands that have changed:

ETE v3 | ETE v4
|:--------------|:-----|
`TreeNode.add_feature()` |`TreeNode.add_prop()`
`TreeNode.add_features()` |`TreeNode.add_props()`
`TreeNode.del_feature()` |`TreeNode.del_prop()`
`TreeNode.<feature>` |`TreeNode.props[<prop>])`
`getattr(TreeNode, <feature>`) |`TreeNode.props.get(<prop>)`
`TreeNode.<feature> = <feature_val>` |`TreeNode.props.add_prop(<prop_val>)`





# The Programmable Tree Drawing Engine

## Overview
ETE's Smartview extension provides a highly programmable drawing system to render any hierarchical tree structure as an interactive graph in the browser.
The produced images could be later exported in SVG format. Although several predefined visualization layouts are included with the default installation, 
custom styles can be easily created from scratch.


Image customization is performed through four elements: 
1. TreeStyle, setting general options about the image (shape, rotation, etc.)
2. NodeStyle, which defines the specific aspect of each node (size, color, background, line type, etc.)
3. Node faces,  which are small pieces of extra graphical information that can be added to nodes (text labels, images, graphs, etc.)
4. A layout function, a normal python function that controls how node styles and faces are dynamically applied to nodes.


The tree can be interactively visualized using a browser-based Graphical User Interface (GUI) invoked by the `TreeNode.explore()` method.



## Interactive visualization of trees

ETE's tree drawing engine is fully integrated with a browser-based GUI, run on your local machine.
Thus, ETE provides means to visualize trees using an interactive interface that allows to explore and manipulate node‚Äôs appearance and tree topology. 
To start the visualization of a node (tree or subtree), you can simply call the `TreeNode.explore()` method. After calling this method, the interactive interface
will be available locally at http://127.0.0.1:5000/ (port 5000 by default) on any desired browser (Google Chrome, Mozilla Firefox...).

All the trees explored in a certain computer would be stored in a **local database**. This feature allows us to seemingly navigate different trees, dynamically change their style, 
topology and export the visualization as an image. 


Argument | Description
|--------------|:-----|
tree_name | name used to store tree in local database. Automatically generated if not provided.
tree_style | tree style applied to the visualized tree. Defaults to TreeStyle class
layouts |  list of layout functions that will be available from the GUI. May be applied at user's discretion
port | port used to run the local server (127.0.0.1). Default 5000


The GUI allows many operations to be performed graphically, however it does not implement all the possibilities of the programming toolkit.


#### Rendering trees as images

While navigating the tree from the GUI, images can be generated and exported in SVG format. This can be done by pressing the key `d` or from the 
interactive control panel (Control panel > Download > svg)




## Customizing the aspect of trees

Image customization is performed through four main elements:

### Tree style

The `TreeStyle` class can be used to create a custom set of options that control the general aspect of the tree image while rendered or displayed as an interactive visualization.
Tree styles can be passed to the `TreeNode.explore()` and `TreeNode.render()` methods.

There are a myriad of features that can be modified.
For instance, `TreeStyle` allows to modify the scale used to render tree branches or choose between circular or rectangular tree drawing modes.


>**NOTE**:
>a number of parameters can be controlled through custom tree style objects, check `TreeStyle` documentation for a complete list of accepted values.


Some common uses include:

#### Show leaf names, branch length, branch support

![ts_attrs](https://github.com/jorgebotas/ete4-documentation/blob/master/ts_attrs.png)

Automatically add node names and branch information to the tree image:

```
from ete4 import Tree
from ete4.smartview import TreeStyle

t = Tree()
t.populate(10)

ts = TreeStyle()

# Modify styling options
ts.show_leaf_name = True
ts.show_branch_length = True
ts.show_branch_support = True

# Trigger the interactive web server
t.explore(tree_name="example", tree_style=ts)
```


### Node style
Through the `NodeStyle` class the aspect of each single node can be controlled, including its size, color, background and branch type.
- A node style can be defined statically and attached to several nodes. There are a couple points to take into consideration:
- If you want to draw nodes with different styles, an independent `NodeStyle` instance must be created for each node.
- Note that node styles can be modified at any moment by accessing the `TreeNode.img_style` attribute.
- Static node styles, set through the `set_style()` method, will be attached to the nodes and exported as part of their information. 
For instance, `TreeNode.copy()` will replicate all node styles in the replicate tree. 
- Note that node styles can be also modified on the fly through a layout function (see layout functions)

![nodestyle](https://github.com/jorgebotas/ete4-documentation/blob/master/nodestyle.png)

Simple tree in which the different styles are applied to each node:

```
from ete4 import Tree, NodeStyle
from ete4.smartview import TreeStyle


t = Tree( "((a:1,b:1),c:1)1:.5;" )

# Basic tree style
ts = TreeStyle()
ts.show_leaf_name = True

def layout_fn(node):
    nstyle = NodeStyle()
    if not node.up:
        # Modify the aspect of the root node
        nstyle["fgcolor"] = "blue"
        nstyle["size"] = 15
        node.set_style(nstyle)
    else:
        # Creates an independent node style for each node, which is
        # initialized with a red foreground color.
        nstyle = NodeStyle()
        nstyle["fgcolor"] = "red"
        nstyle["size"] = 5
        node.set_style(nstyle)

layout_fn.__name__ = "Custom node style"
ts.layout_fn = layout_fn

t.explore(tree_name="example", tree_style=ts)
```


### Node faces

Node faces are small pieces of graphical information that can be linked to nodes. 
For instance, text labels or external images could be linked to nodes and they will be plotted within the tree image.

Several types of node faces are provided by the main `ete4` module, 
ranging from simple text (`TextFace`) and geometric shapes (CircleFace), 
to molecular sequence representations (SequenceFace), 
heatmaps and profile plots (ProfileFace). 
A complete list of available faces can be found at the ete4.smartview reference page.

>**NOTE**:
> we are still working on migrating ETE v3 Faces to ETE v4's drawing engine


#### Face position

Faces can be added to different areas around the node, namely **branch-right**, **branch-top**, **branch-bottom** or **aligned**.
Each area represents a table in which faces can be added through the `TreeNode.add_face()` method. 
For instance, if two text labels want to be drawn bellow the branch line of a given node,
a pair of TextFace faces can be created and added to the columns 0 and 1 of the branch-bottom area:

```
from ete4 import Tree
from ete4.smartview import TreeStyle, TextFace

t = Tree( "((a:1,b:1),c:1)1:.5;" )

# Basic tree style
ts = TreeStyle()
ts.show_leaf_name = False
ts.show_support = False

# Add two text faces to different columns
def layout_fn(node):
    if node.is_leaf():
        node.add_face(TextFace("hello", padding_x=15), 
                column=0, position="branch-right")
        node.add_face(TextFace("world!"), 
                column=1, position="branch-right")

layout_fn.__name__ = "Hello world"
ts.layout_fn = layout_fn

t.explore(tree_name="example", tree_style=ts)
```

![hello world](https://github.com/jorgebotas/ete4-documentation/blob/master/hello_world.png)


If you add more than one face to the same area and column, they will be piled up.
See the following image as an example of face positions:

[ img and source code ]


>**NOTE**:
> once a face object is created, it can be linked to one or more nodes. For instance, the same text label can be recycled and added to several nodes.



#### Face display


Due to the fact that the interactive visualization tool allows to zoom in and out of large phylogenies,
nodes are dynamically showed and collapsed. Therefore, different faces could be applied to the same node depending 
on whether it is collapsed or not. 


>**NOTE**:
> a node collapses when its children are too small to be seen. 
> The collapsing threshold can be easily modified by the user while navigating the GUI.


In order to add faces to a node in these two different states, collapsed or not, 
an additional parameter has to be passed to the `TreeNode.add_face()` method: `collapsed_only`.

For example, let us pretend we have a large tree and want to know how many leaves are under each node.
For this, we create a layout function which will add a `TextFace` with the number of leaves 
(length of the node) to each internal node and only show it when it is collapsed (no children are shown).
Thus, we can zoom in and out the graph, collapsing nodes and dynamically displaying the number of leaves under each one.

```
from ete4 import Tree
from ete4.smartview import TreeStyle, TextFace


t = Tree()
t.populate(10000)


ts = TreeStyle()
# Remove leaf name.
# We want to display the number of leaves
ts.show_leaf_name = False


def layout_fn(node):
    if node.is_leaf():
        # Leaves will have an arbitrary value of 1
        node.add_face(TextFace("1"), position="branch-right", column=0)
    else:
        nleaves = len(node)
        # Display face ONLY when its children are not visible (collapsed)
        node.add_face(TextFace(str(nleaves)), position="branch-right",
                      column=1, collapsed_only=True)

# Provide descriptive name (later seen in GUI)
layout_fn.__name__ = "Number of leaves"


# Add layout function to TreeStyle 
# so it will be automatically applied in the GUI
ts.layout_fn = layout_fn


t.explore(tree_name="example", tree_style=ts)
```

![nleaves](https://github.com/jorgebotas/ete4-documentation/blob/master/nleaves.png)

> **NOTE**:
> provide the layout function with an descriptive name, as this will be the name with which 
> this layout function is later displayed in the GUI and interactively switched on and off.
> 
> To do so assign an appropriate `__name__` attribute to the layout function as in the example above.

>**NOTE**:
> when adding a faces to a collapsed node in "branch-right" position (`collapsed_only` flag set to True), the first available column is number `1`,
> as column `0` is occupied by the `OutlineFace`, characteristic of collapsed nodes.




### Layout functions

Layout functions act as pre-drawing hooking functions. 
This means, when a node is about to be drawn, it is first sent to a layout function. 
Node properties, style and faces can be then modified on the fly and return it to the drawer engine. 
Thus, layout functions can be understood as a collection of rules controlling how different nodes should be drawn.

Moreover, these layout functions act as styling layers which can be combined both programmatically from a python script as well as
interactively from the GUI.

There are some points to take into consideration:
- Layout functions must have an ***descriptive and unique name*** to be indexed and displayed in the GUI. 
  This is key or the layout functions will override each other when sent to the front-end.
  To assign such name, use the `__name__` attribute. *E.g.* `layout_fn.__name__ = "Some descriptive and unique name"`
- When a layout function adds a face in aligned position, we must declare so. This is particularly necessary 
  when visualizing rectangular representations, as the aligned faces reside in an independent aligned panel which should be triggered.
  In a similar manner, the layout function accepts a `contains_aligned_face` attribute. Thus, assign this flag a truthy value after 
  declaring the function, *i.e.* `layout_fn.contains_aligned_face = True`

Due to the necessity of these two elements, we suggest one of many ways of writing layout functions: ***layout function getters***. 
These will allow us to encapsulate our code more cleanly and reuse it in different projects. The main structure of these getters is:
```
def layout_fn_getter():
    def layout_fn(node):
        # style node as wanted and maybe add a couple of Faces
    
    # Provide the layout function a descriptive and unique name
    layout_fn.__name___ = "My first layout function"
    
    # If the layout function adds faces in aligned position, declare so:
    layout_fn.contains_aligned_face = True or False
    
    # Return the layout function
    return layout_fn


# Later feed the layout function returned by the getter to EITHER:

# the TreeStyle => layout function will be automatically applied
ts = TreeStyle()
ts.layout_fn = [ layout_fn_getter() ]

# the "layout function stack" => layout function will NOT be automatically applied
layouts = [ layout_fn_getter() ]

# Launch the explorer
tree.explore(tree_name="example", tree_style=ts, layouts=layouts)
```

When providing the drawing engine with layout functions, these can be added either to:
- The TreeStyle and therefore, be applied to the explored tree automatically when launching the GUI.
- The "layout function stack" and therefore, the layout function will be accessible from the front-end GUI
  to be switched on and off, but it will be turned off by default.
    
Notice that in the previous example we have added the same layout function both to the TreeStyle and the "layout function stack"
by means of illustration, however, this is NOT recommended. 
We should provide each layout function only to one of the previous two. 
Nonetheless, this will not raise an error as the layout function in the stack 
will be simply overridden by the one in the TreeStyle (with the same name).


## Smartview. Navigating the GUI

ETEv4's Graphical User Interface (GUI) can be run locally and displayed in any desired browser.
To do so, launch the Smartview server:
- By calling the `TreeNode.explore()` method from a Python script to visualize a particular tree of interest.
- By running `python ete4/smartview/gui/server.py` from ETE's root directory. 
  Then, open your browser at http://127.0.0.1:5000/. 
  A welcoming page will be prompted in order for you to upload your trees.


Here is the example tree built in the previous section:

![gui](https://github.com/jorgebotas/ete4-documentation/blob/master/gui.png)


As it can be seen, three main components comprise the GUI:

- The **canvas**: where our visualization is displayed.
- The **control panel** (top right corner): from where to perform a myriad of actions ranging from 
  changing the displayed tree to turning layout functions on and off, downloading an image of the 
  visualization or converting the shown tree to ultrametric.
- The **minimap** (bottom right corner): which represents the extension of the current viewport with respect to the whole tree.
  
  
### The canvas
  
  
Here is where the tree is visualized. By scrolling we can zoom in-and-out a particularly node, adapting the viewport 
to it [^smartscroll], while dynamically showing or collapsing nodes depending on the zoom level. 
The graph can be intuitively translated by clicking and dragging.
Actions can be performed on a particular node, by right-clicking on it, a menu with options will be prompted.
[^smartscroll]: The "smart scroll" feature can be switched to regular scroll from the control panel.

![contextmenu](https://github.com/jorgebotas/ete4-documentation/blob/master/contextmenu.png)



#### Aligned panel


Faces in aligned position (see *Face position*) are displayed in a so-called aligned panel which 
can be dragged along the horizontal axis. Let us see an example where we can visualize 
a Multiple Sequence Alignment in this panel.



### The control panel

Three tabs comprise the control panel:

1. Tree view: general information about the visualization. Here we can see the name of the displayed tree (value passed to `TreeNode.explore()`),
   and if we click on it, a list of all the trees stored in the local database, from which to choose, will be prompted. 
   In this section we can also sort the tree, upload a new one (welcome page), download (newick or image) 
   or obtain and modify information related to be visualization.
    
    ![cp_view](https://github.com/jorgebotas/ete4-documentation/blob/master/cp_view.png)

2. Representation: options pertaining to the Smartview drawer and layout functions. 
   Here we can change the type of drawer (rectangular or circular), 
   the collapsing threshold (default 15px), convert the displayed tree to ultrametric, and 
   combine all the layout functions provided through the TreeStyle and the "layout function stack"
   in a mix-and-match fashion.

    ![cp_rep](https://github.com/jorgebotas/ete4-documentation/blob/master/cp_rep.png)

3. Selection: options related to manually collapsed, tagged and searched nodes. To toggle a search simply type `/`.
    
    ![cp_sel](https://github.com/jorgebotas/ete4-documentation/blob/master/cp_sel.png)
