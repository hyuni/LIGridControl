LIGrid
=============

An efficient variable-sized grid of NSCells. LIGrid supports Mac OS 10.9 Mavericks and later. To use LIGrid, import the contents of the **Classes** folder into your own project. To see a sample of LIGrid usage, open LIGridControl.xcodeproj and run the sample application which draws a custom grid.

Features
--------

LIGrid is an alternative to NSTableView that provides more efficient support for variable sized rows and columns, extensive support for grid layout and styling, and keyboard control familiar to spreadsheet users.

Classes
-------

LIGridControl contains both Objective C and C++ classes. The C++ implementation serves as a kernel of sorts for layout logic and has been separated from the Objective C portion so that it can be reused in an iOS implementation of the grid. The Mac version of LIGrid is implemented using the NSController-NSCell system for best performance while a future iOS implementation will use layers.

C++ classes in the project live in the **li::** namespace and include:

- **geom::point** and **geom::rect** - CGPoint and CGRect-like classes which are interchangeable with their CG... equivalent structures, but which add logic like intersection, union, and containment tests.

- **grid::span** - Span represents the starting point and size of a row, column, row divider, or column divider. Grid layouts are, effectively, arrays of spans for the row (y) and column (x) axes.

- **grid::interval** - represents a range of cells or spans along the row or column axis. The interval class provides functions to convert between cell intervals and span intervals.

- **grid::area** - Area represents a grid cell area: either a single (row, column) pair, or a range of rows and columns. Areas use interval objects to represent ranges of rows and columns and express their ranges in terms of cells rather than spans. for N cells along either the row or column axis, 2N + 1 spans exist: 

		div(0) : cell(0) : div(1) : cell(1) ... div(N) : cell(N) : div(N+1)

- **grid::grid** - a grid layout which stores row, column, and divider sizes. grid also stores fixed grid areas to represent cells that have been joined together and that are tagged with associated Objective C objects. grid is our layout kernel.

Objective C classes in the project implement grid visuals and event handling:

- **LIGrid** - a grid of cells and dividers. In a spreadsheet style layout, LIGrid is used to represent both the spreadsheet proper and its associated row and column headers. LIGrid defines both a data source and delegate protocol used to populate grid data and modify how that data is displayed within the grid.

- **LIGridArea** - a cell area that corresponds either to a single row:column pair, or to a range of rows and columns and an associated Objective C object if the area is fixed. 

- **LIGridSelection** - an object used to represent grid selection. Grids can have multiply-selected cells and ranges of cells. LIGridSelection represents each distinct selection in the control, and has methods used to extend selection or to move it. Selection in grids whose cells are all single row:column pairs is a pretty simple matter; but grids with cells that span multiple rows and columns complicate selection logic. LIGridSelection encapsulates and abstracts this complication.

- **LIGridFieldCell**, **LIGridDividerCell** - cells used to display grid cell data and dividers. If you want to change the look of LIGrid, these are the classes you need to work with or possibly subclass. Associated NSControls for each are included in the project mostly as a convenience - you may want to display a cell or divider outside of a grid (in an inspector, for example) and these controls are how you do it. 


- **LITable** - a collection of LIGrids: a central grid bordered by row and column header grids. LITable can act either as the documentView, or subview of the documentView of an NSScrollView. LITable floats its row and column headers when they approach the edge of the visible scrolling area.

- **LIShadow** - a drop shadow used to border floating LITable row and column headers.

NSCells vs. NSViews
-------------------

LIGridControl is an evolution of some working code in an as-yet unreleased Mac app. The original rationale for the project was to experiment with creating a view-based grid versus the original grid's cell-based display.

After profiling, I found that NSViews were too heavy for the sort of work LIGridControl was trying to do - window zooming in particular felt far too heavy. I've reverted to my NSCell based implementation and repurposed LIGridControl as a cleanup of my original grid code. Layout and display - expressed as C++ and Objective C classes - are now properly separated.

Eventually grid will move to iOS where an analog of NSCell doesn't exist. LIGridControl's C++ layout classes will be used in the iOS port with UIView-based "cells." In the iOS world at least, window zooming isn't a problem.

Key Event Handling
------------------

LIGrid defines a block property executed on keyDown: and assigns a default implementation of the block consistent with typical spreadsheet key handling.

The block initiates editing if an alphanumeric or punctuation character is keyed. If the '=' sign is keyed, then a new optional responder method **insertFunction:** is passed through the responder chain.

At time of writing, the block is implemented like so:

    __weak LIGrid *weakSelf = self;
    _keyDownHandler = ^BOOL(NSEvent *keyEvent) {
        if ([keyEvent.characters isEqualToString:@"="]) {
            [weakSelf doCommandBySelector:@selector(insertFunction:)];
            return YES;
        } else {
            NSMutableCharacterSet *editChars = [[NSCharacterSet alphanumericCharacterSet] mutableCopy];
            [editChars formUnionWithCharacterSet:[NSCharacterSet punctuationCharacterSet]];
            
            if (weakSelf.selectedArea != nil) {
                if ([[keyEvent characters] rangeOfCharacterFromSet:editChars].location != NSNotFound) {
                    [weakSelf editGridArea:weakSelf.selectedArea];
                    [weakSelf.currentEditor insertText:keyEvent.characters];
                    return YES;
                }
            }
        }
        return NO;
    };

Please refer to LIGrid.mm for the most recent implementation of the key handler block.

License & Notes
---------------

LIGrid is licensed under the MIT license and hosted on GitHub at https://github.com/monyschuk/LIGridControl/. Fork the project and feel free to send pull requests with your changes!

TODO
----

In my haste to get this out, some larger bits have been left TBD (to be done):

* ~~LITable with header, footer, and content grids~~
* LITableLayout that orchestrates header, footer, and content grids

Other areas of code that relate to LIGrid and that need working out include:

* extended keyboard controls
* collapsed row and column support
* row and column divider dragging and related delegate methods
* LIGridDividerCell is stubbed, double strokes and dashes need to be implemented

NOTE
----

Please note that my principal motivation in opensourcing this class was to get feedback and fixes from others who need this sort of control. If you find this code useful but spot anything that's broken, or if you find something that's implemented poorly, please feel free to fork and submit a pull request. This code will eventually make its way into some commercial software and I'd like as many eyes on it as I can find. If you'd like to contact me directly about this code, email me at mark.onyschuk@gmail.com or find me on Skype at monyschuk.

