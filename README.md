
### | PyETL
_python3.5 package for ETL jobs_

***


### Install
```sh
# create a python3.5 environment and install the package
$ pip install -i https://test.pypi.org/simple/ PyETL==1.0.8
# start the python shell and execute the test command
# if the test command completed successfully, 
# proceed with the Quickstart
$ python
>>> import PyETL
```

### Quick Start
```sh
# pre-built pipeline: transform
>>> from PyETL import load
>>> from PyETL import transform
>>> raw_data = load("/Users/abra.cadabra/data/Challenge_me.txt")
>>> transformed_data = transform(raw_data)
>>> transformed_data
```

```sh
# custom pipeline
>>> from PyETL import Utils
>>> from PyETL import Transformer
>>> from PyETL import load
>>> raw_data = load("/Users/abra.cadabra/data/Challenge_me.txt")
>>> line_objects = Utils.linearize(raw_data)
>>> transformer = Transformer()
>>> transformed_data = transformer\
        .fit(line_objects)\
        .dropna()\
        .encode(['engine-location'], encoder='integer_encoder')\
        .str_2_int(['num-of-cylinders'])\
        .flag({'aspiration': 'turbo'})\
        .enforce_type()\
        .scale({'price': 0.01})\
        .collect()
>>> transformed_data
```

### Architecture
+ Data file is read and each line is stored in a list producing `list of strings`
+ `Class Line` wraps each line in the list and creates a `list of line objects`
+ Extracting specified column values and flagging `line objects` containing null values are done on the fly during the initialization phase of the line objects
+ Transformations are applied on the `list of line objects`
+ Transformers utilize `map()`, `filter()` and `lambda` functions for performance
+ Transformers are chainable via the help of `decorators` to achieve flexibility and generalization e.g. `transformer.func1().func2().func3()`
+ Integer encoding is done with the help of `generators`

### Task
ETL pipeline that:
+ extracts data from a given file (.txt),
+ transforms the data
+ and returns a matrix (list of lists).

### Schema
| Field | Type |
| ------ | ----------- |
| engine-location   | int |
| num-of-cylinders | int |
| engine-size    | int |
| weight    | int |
| horsepower    | float |
| aspiration    | int |
| price    | float |
| make    | byte (utf-8 encoded str) |

### Reference
+ **Utils** - class Utils()
  - Collection of the helper classmethods 
  - Methods:
    * read_file(cls, filename)
    * extract_headers(cls, raw_data)
    * extract_data(cls, raw_data)
    * parse_headers(cls, header_row, header_delimiter=";")
    * map_columns(cls, original_columns)
    * linearize(cls, raw_data)
    * int_code_generator(cls)
    * integer_encoder(cls, val, generator)
+ **Line**  - class Line(raw_line, col_indices)
  - Wrapper class for a raw line read from the data file
  - Implements transformers for each line 
  - Args:
    * raw_line: str, a row from the data file
    * col_indices: list of int, indices of the specified columns in the data file
  - Returns:
    * line object
  - Properties:
    - raw_line: same as in Args
    - col_indices: same as in Args 
    - parsed_raw_line: default=None, extracted specified column values stored in a list, line transformations mutate this property as the transformers apply
    - has_null: default=False, if any of the specified columns is null 
  - Methods:
    * _parse(self, field_delimiter=";")
    * _check_null(self, null_char='-')
    * _flag(self, params)
    * _encode(self, fields, encoder='integer_encoder')
    * _str_2_int(self, fields)
    * _enforce_type(self)
    * _scale(self, params)
    * _process_line(self)
    * repr(self)
+ **Transformer**  - class Transformer()
   - Implements transformer methods that are applied on the list of line objects
   - Returns:
     * transformer object
   - Properties:
     * orig_data: default=None, list of line objects, before the specific transformation
     * transformed_data: default=None, list of line objects, after the specific transformation
     * Note1: orig_data -> (transformer) -> transformed_data
     * Note2: via the `@chain` decorator, if the transformer in chained, i.e. transformer is applied on the output of the previous transformer, orig_data becomes the transformed_data, so that the new transformation can be applied on the previously-transformed data
    - Methods:
      * set_orig_data(self, orig_data)
      * set_transformed_data(self, transformed_data)
      * fit(self, data)
      * chain(func)
      * dropna(self)
      * encode(self, fields, encoder='integer_encoder')
      * str_2_int(self, fields)
      * flag(self, params)
      * enforce_type(self)
      * scale(self, params)
      * collect(self)
      * copy(self)
+ **load**  - def load(dataFile)
   - Reads an input file and generates a list of strings representing each line
   - Args:
     * dataFile: str, Location of the .txt file e.g. ~/Challenge_me.txt
   - Returns:
     * list of strings 
+ **transform**  - def transform(raw_data)
  - A pre-built pipeline that achieves the task
  - Args:
    * raw_data: list of strings, each string representing a row from the data file
  - Returns:
    * transformed data: list of list, transformed data where the first entry is the list of headers
  - Stages of transformations:
    * dropna(): dropns the entry if any of the specified columns is null 
    * encode(['engine-location'], encoder='integer_encoder'): integer encoder i.e. 'green: 0, yellow: 1, black: 2', converts labels to integers, can be decoded to recover the original data via the internal map
    * str_2_int(['num-of-cylinders']): 'three' -> 3
    * flag({'aspiration': 'turbo'}): if 'turbo', flag it as 1, 0 otherwise
    * enforce_type(): converts the specified column data types as described in the schema 
    * scale({'price': 0.01}): scales the specified field
    * collect(): collects the payload from the transformed line objects and inserts the headers as a first entry


