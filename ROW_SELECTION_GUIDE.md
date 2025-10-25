# Row Selection and Scrolling Guide for DataTable2

## Overview
This guide explains how to use the new row selection and scrolling functionality in `DataTable2`. You can now select rows by index and automatically scroll to them with smooth animations.

## New Properties

### `selectedRowIndex`
- **Type**: `int?`
- **Description**: The index of the currently selected row (0-based). Set to `null` if no row is selected.
- **Default**: `null`

### `onRowSelected`
- **Type**: `ValueChanged<int>?`
- **Description**: Callback triggered when a row is selected by index through the `scrollToRow()` method.
- **Default**: `null`

## New Methods

### `scrollToRow()`
Scrolls to a specific row by index and optionally selects it.

```dart
void scrollToRow(
  int rowIndex, {
  Duration duration = const Duration(milliseconds: 300),
  Curve curve = Curves.easeInOut,
  bool shouldSelect = false,
})
```

**Parameters:**
- `rowIndex`: The index of the row to scroll to (0-based)
- `duration`: Animation duration for scrolling (default: 300ms)
- `curve`: Animation curve for the animation (default: easeInOut)
- `shouldSelect`: If true, triggers the `onRowSelected` callback

### `isRowSelected()`
Checks if a specific row is currently selected.

```dart
bool isRowSelected(int rowIndex)
```

**Parameters:**
- `rowIndex`: The index of the row to check

**Returns**: `true` if the row is selected, `false` otherwise

## Usage Example

```dart
import 'package:flutter/material.dart';
import 'package:data_table_2/data_table_2.dart';

class MyDataTableScreen extends StatefulWidget {
  @override
  _MyDataTableScreenState createState() => _MyDataTableScreenState();
}

class _MyDataTableScreenState extends State<MyDataTableScreen> {
  late ScrollController _scrollController;
  int? _selectedRowIndex;

  @override
  void initState() {
    super.initState();
    _scrollController = ScrollController();
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  void _selectAndScrollToRow(int rowIndex) {
    setState(() {
      _selectedRowIndex = rowIndex;
    });
    
    // Scroll to the row programmatically
    // Access the DataTable2 state or store reference to scroll
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Data Table with Row Selection'),
      ),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(16.0),
            child: TextField(
              decoration: InputDecoration(
                labelText: 'Enter row index to select',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (value) {
                final index = int.tryParse(value);
                if (index != null) {
                  _selectAndScrollToRow(index);
                }
              },
            ),
          ),
          Expanded(
            child: DataTable2(
              scrollController: _scrollController,
              selectedRowIndex: _selectedRowIndex,
              onRowSelected: (index) {
                print('Row $index selected');
              },
              columns: [
                DataColumn2(label: Text('Name')),
                DataColumn2(label: Text('Email')),
                DataColumn2(label: Text('Age')),
              ],
              rows: List<DataRow2>.generate(
                50,
                (index) => DataRow2(
                  selected: _selectedRowIndex == index,
                  onTap: () {
                    setState(() {
                      _selectedRowIndex = index;
                    });
                  },
                  cells: [
                    DataCell(Text('Person $index')),
                    DataCell(Text('person$index@example.com')),
                    DataCell(Text('${20 + (index % 50)}')),
                  ],
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

## Advanced Usage with ScrollController

For more control, you can access the `scrollToRow()` method directly:

```dart
// Get reference to DataTable2 widget state
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final GlobalKey<_DataTableState> _tableKey = GlobalKey();
  
  void _jumpToRow(int rowIndex) {
    _tableKey.currentState?.scrollToRow(
      rowIndex,
      duration: Duration(milliseconds: 500),
      curve: Curves.easeInOutCubic,
      shouldSelect: true,
    );
  }

  @override
  Widget build(BuildContext context) {
    return DataTable2(
      // ... properties
    );
  }
}
```

## Highlighting Selected Rows

To visually highlight the selected row, use conditional styling:

```dart
rows: List<DataRow2>.generate(
  100,
  (index) => DataRow2(
    selected: _selectedRowIndex == index,
    color: _selectedRowIndex == index 
      ? WidgetStateProperty.all(Colors.blue.withOpacity(0.1))
      : null,
    onTap: () {
      setState(() {
        _selectedRowIndex = index;
      });
    },
    cells: [
      DataCell(
        Text(
          'Row $index',
          style: _selectedRowIndex == index
            ? TextStyle(fontWeight: FontWeight.bold, color: Colors.blue)
            : null,
        ),
      ),
      // ... more cells
    ],
  ),
)
```

## Performance Considerations

- **Scroll Position**: The scroll position is calculated based on `dataRowHeight` and the number of header rows.
- **Large Datasets**: For tables with thousands of rows, consider using pagination or virtual scrolling.
- **Animation Duration**: Adjust the duration parameter based on your UX preferences.
- **ScrollController**: Always dispose of the `ScrollController` in the `dispose()` method.

## Notes

- The `selectedRowIndex` is independent of row selection via checkboxes; they work in parallel.
- The `scrollToRow()` method only works if a `scrollController` is provided to `DataTable2`.
- Row indices are 0-based (first row is index 0).
- The scroll position accounts for the heading row height automatically.

## Examples

### Scroll to next/previous row
```dart
void _scrollToPreviousRow() {
  if (_selectedRowIndex != null && _selectedRowIndex! > 0) {
    _selectAndScrollToRow(_selectedRowIndex! - 1);
  }
}

void _scrollToNextRow() {
  if (_selectedRowIndex != null && _selectedRowIndex! < rows.length - 1) {
    _selectAndScrollToRow(_selectedRowIndex! + 1);
  }
}
```

### Keyboard navigation
```dart
@override
Widget build(BuildContext context) {
  return Focus(
    onKey: (node, event) {
      if (event.isKeyPressed(LogicalKeyboardKey.arrowUp)) {
        _scrollToPreviousRow();
        return KeyEventResult.handled;
      } else if (event.isKeyPressed(LogicalKeyboardKey.arrowDown)) {
        _scrollToNextRow();
        return KeyEventResult.handled;
      }
      return KeyEventResult.ignored;
    },
    child: DataTable2(
      // ... properties
    ),
  );
}
```
