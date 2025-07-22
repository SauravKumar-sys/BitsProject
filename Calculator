import math

def add(x, y):
    """Adds two numbers."""
    return x + y

def subtract(x, y):
    """Subtracts two numbers."""
    return x - y

#def multiply(x, y):
    """Multiplies two numbers."""
    return x * y

def divide(x, y):
    """Divides two numbers. Handles division by zero."""
    if y == 0:
        return "Error: Division by zero!"
    return x / y

def power(x, y):
    """Calculates x raised to the power of y."""
    return x ** y

def square_root(x):
    """Calculates the square root of a number. Handles negative input."""
    if x < 0:
        return "Error: Cannot calculate square root of a negative number!"
    return math.sqrt(x)

def log_base10(x):
    """Calculates the base-10 logarithm of a number. Handles non-positive input."""
    if x <= 0:
        return "Error: Logarithm input must be positive!"
    return math.log10(x)

def natural_log(x):
    """Calculates the natural logarithm (base e) of a number. Handles non-positive input."""
    if x <= 0:
        return "Error: Logarithm input must be positive!"
    return math.log(x)

def sin_val(x):
    """Calculates the sine of an angle (in radians)."""
    return math.sin(x)

def cos_val(x):
    """Calculates the cosine of an angle (in radians)."""
    return math.cos(x)

def tan_val(x):
    """Calculates the tangent of an angle (in radians). Handles cases where tan is undefined."""
    try:
        return math.tan(x)
    except ValueError: # math.tan can raise ValueError for multiples of pi/2
        return "Error: Tangent undefined for this angle."


def calculator():
    """
    An advanced command-line calculator that performs basic and advanced
    arithmetic and trigonometric operations.
    """
    print("Advanced Calculator")
    print("Select operation:")
    print("1. Add")
    print("2. Subtract")
    print("3. Multiply")
    print("4. Divide")
    print("5. Power (x^y)")
    print("6. Square Root")
    print("7. Logarithm (base 10)")
    print("8. Natural Logarithm (ln)")
    print("9. Sine")
    print("10. Cosine")
    print("11. Tangent")
    print("12. Exit")

    while True:
        choice = input("Enter choice(1/2/3/4/5/6/7/8/9/10/11/12): ")

        if choice in ('1', '2', '3', '4', '5'): # Operations requiring two numbers
            try:
                num1 = float(input("Enter first number: "))
                num2 = float(input("Enter second number: "))
            except ValueError:
                print("Invalid input. Please enter numbers only.")
                continue

            if choice == '1':
                print(f"{num1} + {num2} = {add(num1, num2)}")
            elif choice == '2':
                print(f"{num1} - {num2} = {subtract(num1, num2)}")
            elif choice == '3':
                print(f"{num1} * {num2} = {multiply(num1, num2)}")
            elif choice == '4':
                result = divide(num1, num2)
                print(f"{num1} / {num2} = {result}")
            elif choice == '5':
                print(f"{num1} ^ {num2} = {power(num1, num2)}")
        
        elif choice in ('6', '7', '8', '9', '10', '11'): # Operations requiring one number
            try:
                num = float(input("Enter number: "))
            except ValueError:
                print("Invalid input. Please enter a number only.")
                continue
            
            if choice == '6':
                result = square_root(num)
                print(f"Square root of {num} = {result}")
            elif choice == '7':
                result = log_base10(num)
                print(f"Log base 10 of {num} = {result}")
            elif choice == '8':
                result = natural_log(num)
                print(f"Natural log of {num} = {result}")
            elif choice == '9':
                result = sin_val(num)
                print(f"Sine of {num} radians = {result}")
            elif choice == '10':
                result = cos_val(num)
                print(f"Cosine of {num} radians = {result}")
            elif choice == '11':
                result = tan_val(num)
                print(f"Tangent of {num} radians = {result}")

        elif choice == '12':
            print("Exiting calculator. Goodbye!")
            break
        else:
            print("Invalid input. Please enter a valid choice (1-12).")
        
        print("\n") # Add a newline for better readability after each operation

if __name__ == "__main__":
    calculator()
