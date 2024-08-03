### 2.1 Numeric Operators
- **Addition (`+`):** Adds two numbers.
  ```java
  int a = 5 + 3; // a = 8
  ```

- **Subtraction (`-`):** Subtracts one number from another.
  ```java
  int b = 10 - 4; // b = 6
  ```

- **Multiplication (`*`):** Multiplies two numbers.
  ```java
  int c = 7 * 2; // c = 14
  ```

- **Division (`/`):** Divides one number by another.
  ```java
  int d = 20 / 4; // d = 5
  ```

- **Modulo (`%`):** Finds the remainder of division.
  ```java
  int e = 10 % 3; // e = 1
  ```

### 2.2 Unary Operators
- **Post-increment (`x++`):** Increases `x` by 1 after its current value is used.
  ```java
  int x = 5;
  int y = x++; // y = 5, x = 6
  ```

- **Pre-increment (`++x`):** Increases `x` by 1 before its value is used.
  ```java
  int x = 5;
  int y = ++x; // y = 6, x = 6
  ```

- **Post-decrement (`x--`):** Decreases `x` by 1 after its current value is used.
  ```java
  int x = 5;
  int y = x--; // y = 5, x = 4
  ```

- **Pre-decrement (`--x`):** Decreases `x` by 1 before its value is used.
  ```java
  int x = 5;
  int y = --x; // y = 4, x = 4
  ```

- **Unary Plus (`+`):** Indicates a positive value (usually redundant).
  ```java
  int a = +5; // a = 5
  ```

- **Unary Minus (`-`):** Negates the value.
  ```java
  int b = -5; // b = -5
  ```

### 2.3 Bitwise Operators
- **Bitwise AND (`&`):** Performs bitwise AND operation.
  ```java
  int a = 5 & 3; // a = 1 (0101 & 0011 = 0001)
  ```

- **Bitwise OR (`|`):** Performs bitwise OR operation.
  ```java
  int b = 5 | 3; // b = 7 (0101 | 0011 = 0111)
  ```

- **Bitwise XOR (`^`):** Performs bitwise exclusive OR operation.
  ```java
  int c = 5 ^ 3; // c = 6 (0101 ^ 0011 = 0110)
  ```

- **Bitwise NOT (`~`):** Inverts all bits.
  ```java
  int d = ~5; // d = -6 (bitwise inversion of 0000 0101)
  ```

- **Shift Left (`<<`):** Shifts bits to the left.
  ```java
  int e = 5 << 1; // e = 10 (0101 << 1 = 1010)
  ```

- **Shift Right (`>>`):** Shifts bits to the right.
  ```java
  int f = 5 >> 1; // f = 2 (0101 >> 1 = 0010)
  ```

- **Unsigned Shift Right (`>>>`):** Shifts bits to the right, fills with zeros.
  ```java
  int g = -5 >>> 1; // g = 2147483642 (unsigned shift of negative number)
  ```

### 2.4 Cast Operators
- **Type Casting (`(type)value`):** Converts a value to a different type.
  ```java
  int a = (int) 5.7; // a = 5 (converts double to int)
  ```

### 2.5 Arithmetic Operators
- **Basic Arithmetic:** Same as Numeric Operators.
- **Division by Zero:**
  ```java
  int a = 5 / 0; // Throws ArithmeticException
  ```

- **Overflow and Underflow:**
  ```java
  int b = Integer.MAX_VALUE + 1; // b = Integer.MIN_VALUE (overflow)
  ```

### 2.6 Comparison Operators
- **Equal (`==`):** Checks if values are equal.
  ```java
  boolean a = (5 == 5); // a = true
  ```

- **Not Equal (`!=`):** Checks if values are not equal.
  ```java
  boolean b = (5 != 3); // b = true
  ```

- **Greater Than (`>`):** Checks if one value is greater than another.
  ```java
  boolean c = (5 > 3); // c = true
  ```

- **Less Than (`<`):** Checks if one value is less than another.
  ```java
  boolean d = (5 < 3); // d = false
  ```

- **Greater Than or Equal To (`>=`):** Checks if one value is greater than or equal to another.
  ```java
  boolean e = (5 >= 5); // e = true
  ```

- **Less Than or Equal To (`<=`):** Checks if one value is less than or equal to another.
  ```java
  boolean f = (5 <= 6); // f = true
  ```

### 2.7 `instanceof` Operator
- **Type Checking (`instanceof`):** Checks if an object is an instance of a specific class.
  ```java
  String s = "hello";
  boolean b = s instanceof String; // b = true
  ```

### 2.8 Boolean Operators
- **Logical AND (`&&`):** Returns true if both conditions are true.
  ```java
  boolean a = (5 > 3 && 4 < 6); // a = true
  ```

- **Logical OR (`||`):** Returns true if at least one condition is true.
  ```java
  boolean b = (5 > 3 || 4 > 6); // b = true
  ```

- **Logical NOT (`!`):** Inverts a boolean value.
  ```java
  boolean c = !(5 > 3); // c = false
  ```

### 2.9 Conditional Operator (`?:`)
- **Ternary Operator:** Returns one of two values based on a condition.
  ```java
  int a = (5 > 3) ? 10 : 20; // a = 10
  ```

### 2.10 Assignment Operators
- **Simple Assignment (`=`):** Assigns a value to a variable.
  ```java
  int a = 5; // a = 5
  ```

- **Compound Assignment (`+=`, `-=`, `*=`, `/=`, `%=`):** Performs an operation and assigns the result.
  ```java
  int b = 10;
  b += 5; // b = 15 (same as b = b + 5)
  ```

- **String Concatenation (`+`):** Concatenates strings.
  ```java
  String s = "Hello " + "World"; // s = "Hello World"
  ```

This summary should help you quickly recall the key operators and their uses!
