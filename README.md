Link to colab : https://colab.research.google.com/drive/10t2TeRRTAWS9DBIiZbBiuM6-x0X-CDWO?usp=sharing

# CSV Reader Implementation

## **Understanding Context Managers**
A **context manager** in Python is a construct that allows for automatic resource management, ensuring that resources like files, network connections, or database connections are properly allocated and deallocated when needed. 

A context manager is most commonly used with the `with` statement in Python, which ensures that resources are cleaned up automatically, even if an error occurs. This helps prevent memory leaks and other resource mismanagement issues.

### **How Context Managers Work**
A class-based context manager typically implements two special methods:
- `__enter__()`: Defines what happens when the `with` block starts.
- `__exit__()`: Defines what happens when the `with` block ends, including cleanup operations.

Using a context manager ensures that resources are managed properly without requiring explicit cleanup code from the user.

---

## **How Our Class-Based Context Manager Works**

### **Step 1: Initialization (`__init__` method)**
```python
 def __init__(self, filename):
        self.filename = filename
        self.file = None
        self.reader = None
        self.named_tuple_class = None
        self.iterator = None
```
- The `__init__` method initializes the class by storing the filename and setting up variables that will later hold the file handle, CSV reader, named tuple class, and iterator.
- The file is **not** opened at this stage to keep initialization lightweight.

### **Step 2: Entering the Context (`__enter__` method)**
```python
 def __enter__(self):
        self.file = open(self.filename, mode='r', newline='', encoding='utf-8')
        sample = self.file.readline()
        self.file.seek(0)
        delimiter = ',' if ',' in sample else ';'  
        self.reader = csv.reader(self.file, delimiter=delimiter)
        headers = next(self.reader)
        self.named_tuple_class = namedtuple("Row", headers)
        self.iterator = iter(self.reader)
        return self
```
- The file is opened in read mode with UTF-8 encoding.
- The first line of the file is read to determine the delimiter (`,` or `;`).
- The `csv.reader` is initialized with the detected delimiter.
- The header row is extracted and used to create a named tuple class for structured data access.
- The CSV file rows are stored in an iterator to support **lazy evaluation**, meaning data is only loaded when needed.
- Finally, `self` is returned so that the context manager can be used in a `with` statement.

### **Step 3: Iteration (`__iter__` and `__next__`)**
```python
 def __iter__(self):
        return self

 def __next__(self):
        return self.named_tuple_class(*next(self.iterator))
```
- `__iter__()` returns `self`, allowing the class instance to be iterable.
- `__next__()` retrieves the next row from the CSV reader and converts it into a named tuple for easy field access.

### **Step 4: Exiting the Context (`__exit__` method)**
```python
 def __exit__(self, exc_type, exc_value, traceback):
        if self.file:
            self.file.close()
```
- Ensures the file is closed when exiting the `with` block, even if an error occurs.
- Prevents resource leaks and ensures proper cleanup.

---

## **Understanding Generators and `contextmanager` Decorator**
Python provides another way to implement context managers using generators and the `@contextmanager` decorator from the `contextlib` module.

### **How a Generator-Based Context Manager Works**
- Uses `yield` instead of `return` to provide values lazily.
- The code before `yield` runs when entering the `with` block.
- The code after `yield` runs when exiting the `with` block, handling cleanup.

### **Step-by-Step Explanation**
```python
@contextmanager
def csv_reader(filename):
    file = open(filename, mode='r', newline='', encoding='utf-8')
    try:
        sample = file.readline()
        file.seek(0)
        delimiter = ',' if ',' in sample else ';'
        reader = csv.reader(file, delimiter=delimiter)
        headers = next(reader)
        Row = namedtuple("Row", headers)
        yield (Row(*row) for row in reader)
    finally:
        file.close()
```
- The file is opened and its delimiter is detected.
- The header row is used to create a named tuple class.
- Instead of returning a value, `yield` provides a generator that produces named tuple rows lazily.
- When the `with` block ends, the `finally` block ensures the file is closed.

---

## **Comparison of Both Approaches**
| Feature | Class-Based Context Manager | Generator-Based Context Manager |
|---------|----------------------------|--------------------------------|
| Uses `__enter__` and `__exit__` | ✅ | ❌ (uses `finally` instead) |
| Uses `__iter__` and `__next__` | ✅ | ❌ (uses `yield`) |
| Returns an object | ✅ (`self`) | ❌ (returns a generator) |
| Requires explicit class creation | ✅ | ❌ |

---

## **How This Project Meets the Goals**

### **Goal 1: Class-Based Context Manager**
- Reads CSV files and extracts headers as named tuples.
- Uses `csv.reader` for structured parsing.
- Implements a proper class-based context manager.
- Ensures lazy iteration using `__iter__` and `__next__`.
- Automatically manages file closing with `__exit__()`.

### **Goal 2: Generator-Based Context Manager**
- Uses `@contextmanager` from `contextlib`.
- Uses `yield` for memory-efficient iteration.
- Implements automatic resource management without explicit `__enter__` or `__exit__`.
- Achieves the same result as Goal 1 in a more concise way.

