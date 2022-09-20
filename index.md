
# Installation

We are working on providing a cleaner way of installing ETE v4. However, as ete4's branch is under active development, we recommend installing the ETE's github repository and building the package "manually"
1. Create a python 3+ environment. An easy way to do this is through [Conda](https://docs.conda.io/en/latest/).

   `conda create -n ete_env && conda activate ete_env`
2. Install dependencies:
    - Cython: recommended installation through [mamba](https://github.com/mamba-org/mamba) or conda-forge
        
        `mamba install cython` or `conda install -c conda-forge cython`
    - Python dependencies
    
        `pip install flask flask-cors flask-httpauth flask-restful flask-compress sqlalchemy numpy PyQt5`

3. Clone ete4 branch

    `git clone -b ete4 https://github.com/etetoolkit/ete ete4 `
6. Build ete4

    `cd ete4 && pip install -e .`

>**NOTE.**
> In Linux there may be some cases where the gcc library must be installed 
> 
> `conda install -c conda-forge gcc_linux-64`

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
`TreeNode.<feature>` |`TreeNode.props[<prop>]`
`getattr(TreeNode, <feature>)` |`TreeNode.props.get(<prop>)`
`TreeNode.<feature> = <feature_val>` |`TreeNode.add_prop()`





# The Programmable Tree Drawing Engine

## Overview
ETE's Smartview extension provides a highly programmable drawing system to render any hierarchical tree structure as an interactive graph in the browser.
The produced images could be later exported in SVG format. Although several predefined visualization layouts are included with the default installation, 
custom styles can be easily created from scratch.


Image customization is performed through four elements: 
1. TreeStyle, setting general options about the image (shape, legend, etc.)
2. NodeStyle, which defines the specific aspect of each node (size, color, background, line type, etc.)
3. Node faces,  which are small pieces of extra graphical information that can be added to nodes (text labels, images, graphs, etc.)
4. A layout class, a python class that controls both changes to the whole tree style as well as how node styles and faces are dynamically applied to nodes.


The tree can be interactively visualized using a browser-based Graphical User Interface (GUI) invoked by the `TreeNode.explore()` method.



## Interactive visualization of trees

ETE's tree drawing engine is fully integrated with a browser-based GUI, run on your local machine.
Thus, ETE provides means to visualize trees using an interactive interface that allows to explore and manipulate node's appearance and tree topology. 
To start the visualization of a node (tree or subtree), you can simply call the `TreeNode.explore()` method. After calling this method, the interactive interface
will be available locally at http://127.0.0.1:5000/ (port 5000 by default) on any desired browser (Google Chrome, Mozilla Firefox...).

>**NOTE**.
> The GUI allows many operations to be performed graphically, however it does not implement all the 
> possibilities of the programming toolkit.

All the trees explored in a certain computer would be stored in a **local database**. This feature allows us to seemingly navigate different trees, dynamically change their style, 
topology and export the visualization as an image. 


When calling the `TreeNode.explore()` method, there are four arguments to consider:

Argument | Description
|--------------|:-----|
tree_name | name used to describe the tree. Automatically generated if not provided
layouts |  list of layout functions that will be available from the GUI. May be applied at user's discretion
show_leaf_name | default `True`
show_branch_length | default `True`
show_branch_support | default `True`
popup_prop_keys | list of node property keys that will be rendered on the node's popup on the browser. Default `["name", "dist", "support", "hyperlink", "tooltip"]`
host | host used to run ETE server. Default 127.0.0.1
port | port used to run ETE server. Default 5000



#### Rendering trees as images

While navigating the tree from the GUI, images can be generated and exported in HTML+SVG or PDF format. This can be done by pressing `d` or from the 
interactive control panel (Control panel > Download > svg | Control panel > Download > pdf)




## Customizing the aspect of trees

Image customization is performed through four main elements:

### Tree style

The `TreeStyle` class can be used to create a custom set of options that control the general aspect of the tree image while rendered or displayed as an
interactive visualization.
Tree style can now ONLY be customized from layout classes. In particular, layout classes have a `set_tree_style(self, tree, tree_style)` which modifies 
the TreeStyle class associated to the tree visualized. Thus, layouts can dynamically modify tree styles.

As of now, tree styles **cannot** be passed to the `TreeNode.explore()` method.

There are a myriad of features that can be modified:

#### Legends

`TreeStyle.add_legend()` allows to create legends for discrete and continous variables:

```
tree_style.add_legend(title="Discrete variable",
       variable="discrete",
       colormap=colormap)

tree_style.add_legend(title="Continuous variable", 
       variable="continuous",
       value_range=(0, 1),
       color_range=("white", "blue"))
```

#### Header and footer
Faces can be added to the aligned panel header and footer from `TreeLayout.set_tree_style()`. The following example adds a scale and a title to the 
aligned panel header and footer respectively.
```
 def set_tree_style(self, tree, tree_style):
     super().set_tree_style(tree, tree_style)

      scale = ScaleFace(width=500, 
              scale_range=(0, 1), 
              formatter='%.2f',
              padding_x=2, padding_y=2)
      text = TextFace("Title", max_fsize=11, padding_x=2)
      
      tree_style.aligned_panel_footer.add_face(scale, column=3)
      tree_style.aligned_panel_header.add_face(text, column=3)
```




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
- A node's style can be modified at any moment by accessing the `TreeNode.img_style` attribute.
- Static node styles, set through the `set_style()` method, will be attached to the nodes and exported as part of their information. 
  For instance, `TreeNode.copy()` will replicate all node styles in the replicate tree. 
- Node styles can be also modified on the fly through a layout function (see layout functions)

Here is a simple tree in which the different styles are applied to each node:

![nodestyle](https://github.com/jorgebotas/ete4-documentation/blob/master/nodestyle.png)

```
from ete4 import Tree
from ete4.smartview import TreeLayout

t = Tree( "((A:1,B:1),C:1)1:.5;" )

class LayoutCustom(TreeLayout):
   def __init__(self, *args, **kwargs):
      super().__init__(name="Custom Layout", *args, **kwargs) # self.name attribute must be set
      
   def set_node_style(self, node):
      nstyle = node.sm_style        
      if not node.up:
         # Modify the aspect of the root node
         nstyle["fgcolor"] = "#ffd34d" # yellow
         nstyle["size"] = 15
      else:
         # node style for descending nodes
         nstyle["fgcolor"] = "#4d79ff" # blue
         nstyle["size"] = 5
    

t.explore(tree_name="example", layouts=[ LayoutCustom ], show_branch_support=False)
```


### Node faces

Node faces are small pieces of graphical information that can be linked to nodes. 
For instance, text labels or external images could be linked to nodes and they will be plotted within the tree image.

Several types of node faces are provided by the main `ete4` module, 
ranging from simple text (`TextFace`) and geometric shapes (`CircleFace`), 
to molecular sequence representations (`SequenceFace`), 
heatmaps and profile plots (`ProfileFace` [^ProfileFace]). 
A complete list of available faces can be found at the `ete4.smartview` reference page.

[^ProfileFace]: coming soon to ETE v4.

>**NOTE**.
> We are still working on migrating ETE v3 Faces to ETE v4's drawing engine.


#### Face position

Faces can be added to different areas around the node, namely **branch-right**, **branch-top**, **branch-bottom** or **aligned**.
Each area represents a table in which faces can be added through the `TreeNode.add_face()` method. 
For instance, if two text labels want to be drawn bellow the branch line of a given node (*e.g.*, node `A`),
a pair of TextFace faces can be created and added to the columns 0 and 1 of the branch-bottom area:

```
from ete4 import Tree
from ete4.smartview import TextFace, TreeLayout

t = Tree( "((A:1,B:1),C:1)1:.5;" )


class LayoutHelloWorld(TreeLayout):
   def __init__(self, *args, **kwargs):
      super().__init__(name="Hello world", *args, **kwargs) # self.name attribute must be set
      
   def set_node_style(self, node):
      # Add two text faces to different columns
      if node.name == "A":
         node.add_face(TextFace("hello", padding_x=15), 
                column=0, position="branch_bottom")
         node.add_face(TextFace("world!"), 
                column=1, position="branch_bottom")


t.explore(tree_name="example", layouts=[ LayoutHelloWorld ], show_branch_support=False)
```

![hello world](https://github.com/jorgebotas/ete4-documentation/blob/master/helloworld.png)


If you add more than one face to the same area and column, they will be piled up.
See the following image as an example of face positions:

![facespos](https://github.com/jorgebotas/ete4-documentation/blob/master/facepos.png)

```
from ete4 import Tree, NodeStyle
from ete4.smartview.ete.layouts import TreeStyle
from ete4.smartview.ete.faces import TextFace, AttrFace

from ete4 import Tree

positions = ["branch-top", "branch-bottom", "branch-right", "aligned"]

t = Tree( "((A:1,B:1),C:1)1:.5;" )

# Basic tree style
ts = TreeStyle()
ts.show_leaf_name = False
ts.show_support = False


# Assign a color to each leaf
colors = ["#78a0ff", "#83aa5a", "#ff7ea5"]
for i, leaf in enumerate(t):
    leaf.add_prop("color", colors[i])

class LayoutFacePositions(TreeLayout):
   def __init__(self, *args, **kwargs):
      super().__init__(name="Face positions", *args, **kwargs) # self.name attribute must be set
      self.aligned_faces = True # Triggers the aligned panel at self.set_tree_style
      
   def set_node_style(self, node):
    if not node.up: # do not style root node
        return

    px, py = 15, 5 # padding_x, padding_y
    for pos in positions:
        if pos == "aligned":
            if node.is_leaf():
                # Set background color to leaf
                ns = node.sm_style
                ns["size"] = 0 # do not show dot
                ns["bgcolor"] = node.props["color"]
                # Add 2x2 matrix
                for col in range(2):
                    node.add_face(TextFace(f'{pos}  row_0  col_{col}',
                        color=node.props["color"]), 
                        column=col, position=pos)
                    node.add_face(TextFace(f'{pos}  row_1  col_{col}',
                        color=node.props["color"]), 
                        column=col, position=pos)
        else:
            # All but aligned position
            text = f'{pos}  row_0  col_0'
            node.add_face(TextFace(text, padding_x=px, padding_y=py),
                    column=0, position=pos)

t.explore(tree_name="example", layouts=[ LayoutFacePositions ], show_branch_support=False)
```


>**NOTE**.
> Once a face object is created, it can be linked to one or more nodes. For instance, the same text label can be recycled and added to several nodes.



#### Face display


Due to the fact that the interactive visualization tool allows to zoom in and out of large phylogenies,
nodes are dynamically showed and collapsed. Therefore, different faces could be applied to the same node depending 
on whether it is collapsed or not. 


>**NOTE**.
> A node collapses when its children are too small to be seen. 
> The collapsing threshold can be easily modified by the user while navigating the GUI.


In order to add faces to a node in these two different states, collapsed or not, 
an additional parameter has to be passed to the `TreeNode.add_face()` method: `collapsed_only`.

For example, let us pretend we have a large tree and want to know how many leaves are under each node.
For this, we create a layout function which will add a `TextFace` with the number of leaves 
(length of the node) to each internal node and only show it when it is collapsed (no children are shown).
Thus, we can zoom in and out the graph, collapsing nodes and dynamically displaying the number of leaves under each one.

```
from ete4 import Tree
from ete4.smartview import TreeLayout, TextFace


t = Tree()
t.populate(10000)

class LayoutLeafNumber(TreeLayout):
   def __init__(self, *args, **kwargs):
      super().__init__(name="Number of leaves", *args, **kwargs) # self.name attribute must be set
      
   def set_node_style(self, node):
      if not node.is_leaf():
        nleaves = len(node)
        # Display face ONLY when its children are not visible (collapsed)
        node.add_face(TextFace(str(nleaves)), position="branch-right",
                      column=1, collapsed_only=True)


t.explore(tree_name="example", layouts=[ LayoutLeafNumber ], show_leaf_name=False)
```

![nleaves](https://github.com/jorgebotas/ete4-documentation/blob/master/nleaves.png)

> **NOTE**.
> Provide the layout class with an descriptive name attribute, as this will be the name with which 
> this layout function is later displayed in the GUI and interactively switched on and off.


>**NOTE**.
> When adding a faces to a collapsed node in "branch-right" position (`collapsed_only` flag set to True), the first available column is number `1`,
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
Actions can be performed on a particular node: right-click on it and a menu with options will be prompted:
[^smartscroll]: The "smart scroll" feature can be switched to regular scroll from the control panel.

![contextmenu](https://github.com/jorgebotas/ete4-documentation/blob/master/contextmenu.png)



#### Aligned panel


Faces in aligned position (see *Face position*) are displayed in a so-called aligned panel which 
can be dragged along the horizontal axis. 
Let us see an example where we can visualize 
a Multiple Sequence Alignment making use of this panel...

[ img and source code ]


### The control panel

Three tabs comprise the control panel:

1. Tree view: general information about the visualization. Here we can see the name of the displayed tree (value passed to `TreeNode.explore()`),
   and if we click on it, a list of all the trees stored in the local database, from which to choose, will be prompted. 
   In this section we can also sort the tree, upload a new one (welcome page), download it (newick or image) 
   or obtain and modify information related to the visualization.
    
    ![cp_view](https://github.com/jorgebotas/ete4-documentation/blob/master/cp_view.png)

2. Representation: options pertaining to the Smartview drawer and layout functions. 
   Here we can change the type of drawer (rectangular or circular), 
   the collapsing threshold (default 15px), convert the displayed tree to ultrametric, and 
   combine all the layout functions provided through the TreeStyle and the "layout function stack"
   in a mix-and-match fashion.

    ![cp_rep](https://github.com/jorgebotas/ete4-documentation/blob/master/cp_rep.png)

3. Selection: options related to manually collapsed, selected and searched nodes. To toggle a search simply press `/`
    
    ![cp_sel](https://github.com/jorgebotas/ete4-documentation/blob/master/cp_sel.png)


## Integrating ETE v4 as a external plug-in in your own project

ETE has the potential of been integrated into external projects, allowing you,
the developer, to control how your website responds to nodes been (un)selected,
and even coordinate additional visualizations with ETE's.


Due to the complex nature of ETE's visualization engine, we have decided to encapsulate all its behaviour inside an 
[iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) in the front end, while in the back end, the server runs
independently from your project. Thus, you can communicate with ETE by means of:
- [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP). Used to upload your newick and start visualizing (POST) or to query for node's information given its id (GET).
- [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage). Used to communicate the iframe and the rest of your front end. A postMessage (`message` event) will be sent to the parent window (by the iframe)
  every time a node is selected or unselected, allowing you to update your GUI accordingly.
  This `message` event has some data associated: 
    - `tid` the tree id you previously provided as the identifier of your uploaded tree (see example bellow for more info).
    - `node` node id (string) used to identify a node within a tree.
    - `name` name used to describe the selected node. E.g. the node's name.
    - `selected` a flag used to determine if the node in question has been selected (`true`) or unselected (`false`).

>**NOTE**.
> The window.postMessage() method safely enables cross-origin communication between a parent window and an iframe within it.

Here, we will provide a very simple example of how to embed ETE in your website as an iframe, listen for node selection and even delete such selection from outside the iframe.

### Uploading a tree and embedding ETE as an iframe

The following code snippet allows you (from your website's javascript) to upload a tree to ETE through a POST method provided four pieces of information:
- `eteUrl` URL where ETE is listening from (your localhost or any server you are running ETE from).
- `name` name used to describe your tree.
- `tid` a numeric tree id (integer) used to identify your tree. Keep in mind that this id must be unique and could be used a security measure, e.g. a hash made from a username in combination with a date or session stamp. 
- `newick` tree in Newick format.

```
// Define the URL in which ete is running
const eteUrl = "http://127.0.0.1:5000";

async function addTree(name, tid, newick) {
    
    // Create form data
    const data = new FormData();
    data.append("id", tid);
    data.append("name", name);
    data.append("newick", newick);

    // Upload tree data through POST
    fetch(`${eteUrl}/trees`, {
        method: "POST",
        headers: {"Authorization": `Bearer hello`}, // important to have headers
        body: data,
    });
    
    // Embed iframe inside a div with id "div_ete" (use different selector if needed)
    div_ete.innerHTML = `
    <iframe src="${eteUrl}/static/gui.html?tree=${tid}" 
            id="ete_iframe"
            width=0
            height=0
            frameBorder=0
            onload="this.width='100%'
                    this.height=screen.height*0.6">
            ${eteUrl}/static/gui.html?tree=${tid}
    </iframe>
    `
}
```

>**NOTE.**
> Uploading a tree through POST can be done from the back end as well (specially useful for large trees), as the only piece of information that the front end needs to visualize it is the **tree id**.

>**NOTE.**
> In this example we have taken for granted that the developer has left an empty div with a descriptive id (`div_ete` in the code snippet above) in their HTML template,
> dedicated to contain ETE's visualization.


### Listening for node selection

As we saw before, embedding ETE in your project and visualizing a tree seemed rather easy, right?. Well, listening for nodes been selected or unselected is no different!.
Just add a `message` event listener to your website's window:


```
async function onPostMessage(event) {
    // Only respond to events from ETE
    if (event.origin != eteUrl)
        return

    const { tid, node, name, selected } = event.data;

    await updateNodes(tid, node, name, selected);

    updateGUI();
}
// Listen for node (un)selection
window.addEventListener("message", onPostMessage);
```


There are a couple things to notice in the code snippet above:
- The if statement restricts postMessages to be originated only from ETE's URL. This allows us to keep our website nicely secured.
- The `updateNodes` and `updateGUI` functions should be written by YOU, as they will control the response of your website to nodes been selected or unselected in ETE's GUI.


A very simple example of `updateNodes` could be:
```
const selectedNodes = [];

async function updateNodes(tid, node, name, selected) {
    if (selected) {  // node has been selected
        const treeNode = tid + "," + node;
        
        // Obtain more node information through GET
        const nodeInfoResponse = await fetch(`${eteUrl}/trees/${treeNode}/nodeinfo`);
        const nodeInfo = await nodeInfoResponse.json();
        
        selectedNodes.push({ node: node, name: name, info: nodeInfo });

    } else  // node has been unselected
        selectedNodes = selectedNodes.filter(n => n.node !== node);
}
```


### Removing node selection

Okay, we are almost there!. Last but not least would be to unselect a node from outside the iframe and notify ETE through a postMessage.

```
ete_iframe.contentWindow.postMessage({ 
    selected: false,
    node: node,  // node id (string)
}, eteUrl);
```

This is everything you need to unselect a node!. In my opinion this is rather beautiful as we do not need to add any additional calls to update our GUI. The iframe itself
will notify its parent window that a node has been unselected, triggering a `message` event, for which we are already listening, and automatically updating the GUI :) !
