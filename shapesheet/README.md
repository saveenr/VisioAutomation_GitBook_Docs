# ShapeSheet

The ShapeSheet is the spreadsheet-like grid of cells that backs every Visio shape, page, master, and document. Almost everything Visio knows about a shape (its size, position, formatting, geometry, behavior, custom data) lives in the ShapeSheet, and most things you can do via the Visio UI ultimately read or write a ShapeSheet cell.

VisioAutomation provides three primary affordances for working with the ShapeSheet:

* **[SRC structs](cells.md)**: how cells are addressed: every ShapeSheet cell is identified by a `(Section, Row, Cell)` triple, modeled in code as the `VisioAutomation.Core.Src` struct. The `VisioAutomation.Core.SrcConstants` static class exposes hundreds of named constants for the well-known cells.
* **[Query the ShapeSheet](query-the-shapesheet.md)**: `CellQuery` and `SectionQuery` batch many cell reads into one Visio call, returning strongly-typed result tables.
* **[Modify the ShapeSheet](modify-the-shapesheet.md)**: `SrcWriter` and `SidSrcWriter` batch many cell writes into one Visio call.

The Query and Writer types are the recommended way to read and write ShapeSheet cells whenever you have more than a handful of cells in play; they are dramatically faster than per-cell access through Visio's COM `Cells[…]` indexer.
