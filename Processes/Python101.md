# Python Notes

## Introduction

This article is still a work in progress as I continue to improve my Python skills.  

Python is platform agnostic, uses natural language, and is extremely powerful. It is currently (2024) my primary programming language due to its versatility and community support.

## Programming101

Line indentation changes the *scope* of the code, you know the drill, 4 spaces no tab.

Data Types:  
    String: alphanumeric characters stored verbatim, typically words and phrases  
    Integer: numerical non decimal value (whole number), ex. 4815162342  
    Float: numerical decimal value, 3.14  
    Boolean: true or false value, 5 == 2 would result in a false value  

Advanced Data Types:  
    Complex: defined and undefined values, x = 83j  
    List: Series of multiple values that can be changed, x = [45, “ex”, 3.2]  
    Tuple: Series of multiple values that cant be changed, x = (45, “ex”, 3.2)  
    Dictionary: A list of pairs, x = {1:”one”, 2:”two”, 3:”three”}  

Define variables with the equals (=) sign. In the following example x, y, and z are all variables that store the integer value of one (1). Ex. x = 1

Loops:
    Recursion: Function calls itself, must code in an escape to prevent infinity  
    Iteration: Processing items in a loop  
    Break: escape loop and leave iteration  
    Continue: escape loop and move to next iteration  

String Methods:  
    Slicing: Only output characters in a range, ex. x[0:10] would display 10 characters of the variable x (which hopefully is a string)  
    Joining: Combine strings and concatenate and add a delineating character, ex . y.join(x)  

## Common Commands

Debugging can be done by putting code inside of a loop consisting of: **try**, **except**, **else**  

Record a user interaction by using input()  

Comments:  
    **””” triple quotes “””**: Create block comments  
    **# pound sign**: Create line comments  

Transforming Data:  
    **str()** - converts to a string  
    **int()** - converts to an integer  

File Operations:  
    **open**: You can read, write, or append a file, ex. Open(sample.txt, r)  
    **file.read**: Limit the read operation to only a portion of the file  
    **file.write**: Write information to the end of the file  
    **file.close**: Releases the file back to the OS  

List Operations:  
    **x.append** - Adds data to the end of list x  
    **x.remove(y)** - Destroys y from list x  
    **x.insert(y,z)** - inserts z at position y in list x  
    **x.index(y)** - Output the position of y in list x  
    **x.extend(y)** - Concatenate list y into list x  
    **len(x)** - Returns number of items in list x  

## Libraries & Packages

[pip](https://github.com/pypa/pip)  - Package manager  
[Conda](https://github.com/conda/conda/)  - Package manager  

### Automation

[Ansible](https://github.com/ansible/ansible)   - Server based deployment using playbooks  
[NAPALM](https://github.com/napalm-automation/napalm)   - Network Automation  
[Celery](https://github.com/celery/celery)   - OS task scheduler  

### AI & ML

[Pandas](https://github.com/pandas-dev/pandas)  - Manipulate tables in python (xlsx/csv formatted data can be used in python)  
[Numpy](https://github.com/numpy/numpy)   - Gives other libraries ability to use arrays and math  
[MatPlotLib](https://github.com/matplotlib/matplotlib) - 2D graphs and charts  
[SciKit-Learn](https://github.com/scikit-learn/scikit-learn) - machine learning library  
[Tensorflow](https://github.com/tensorflow/tensorflow)  - Powerful AI framework  
[PyTorch](https://github.com/pytorch/pytorch)  - NumPy but GPU accelerated  
[LangChain](https://github.com/langchain-ai/langchain)  - Natural Language Processing  
