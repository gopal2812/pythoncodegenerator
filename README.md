# python code generator
## Dataset
I have downloaded the CodeSerNet Data(https://github.com/github/CodeSearchNet) for initial training. I have taken the cleaned code of small size (200) for training the model and then train the model with the assignment data.For training both the I have kept the vocabulary of assignment data for both the training sample.In the Assignment data:There are some 4600+ examples of English text to python code and From the CodeSearchNEt data Added the another 6K small samples for training.I have then converted the list into dataframe and then saved the file as CSV to use tabulardataset function with CSV file extension. I have used a dataset iterator for training and validation purposes. First I train and validate my model in a combined dataset. I pick the best model and then used it as a initial model for assignment dataset.It has improved the overall perplexity

## Data Preparation/preprocessing Strategy
a) Ran the pep8 command to align the data and fix some of the code automatically.
b) Ran yapf tool to automatically fix some of the compilation issue.This tool helps in solving a lot of issues and indicating where the issues are.
pip install yapf
yapf your_script.py    # dry-run, only print
Some simple bash scripting also helped in cleaning the data.
i)!cat  /content/drive/MyDrive/clean_data.txt | grep "def " | grep -v ":"
ii)!cat  /content/drive/MyDrive/clean_data.txt | grep -in "print " | grep -v "(" | grep -v "#"
iii)fix print ( errors
iv)fix braces errors
v)To fix compilation error related to 2.7 -3 conversion like print() and others
vi)braces error
vii) colon missing
viii) Remove the # after hash
!cat /content/drive/MyDrive/clean_data_ex.txt | grep  -in "#" | grep " #"
ix)sep = ' #'
stripped = text.split(sep, 1)[0]
#yapf -i your_script.py # replace content
c) Added the python function to remove the comment between the code (def remove_code_comments)
d) Added  the dataframe logic has given me a two list and they are Code and Description


## Embedding Strategy
I did some experimentation on Tokenizer and Embedding. I used the separated tokenizer for code and Description. 
i) For Description I used the spacy en_core_web_trf.
ii)For Code embedding, I tried with two tokenizers. In the final model spacy symbol has provided better handling. I also tried AST as mentioned in the link but failed in implementing it fully due to the timeline. (https://www.asmeurer.com/brown-water-python/alternatives.html). I also explored cubert but couldn't 
work much on it.(https://onevara.com/j143/google-research/-/blob/f869403f20e306aee61d6ed094df205f3f677ca8/cubert/python_tokenizer.py)
Embedding Used in Model:
a) Python tokenizer with special character handling
#https://docs.python.org/3/library/tokenize.html
import io
from io import BytesIO
from tokenize import tokenize, untokenize, NUMBER, STRING, NAME, OP, tok_name

def tokenize_python(code_snippet):
    tokens = tokenize(io.BytesIO(code_snippet.encode('utf-8')).readline)
    parsed = []
    for token in tokens:
        if token.type not in [0,59,60,61,62,63,256]:
            parsed.append(token.string)
    return parsed
b) Spacy English tokenizer with special character handling


## HyperParameter:
i) Increased DEC_PF_DIM , ENC_LAYER and drop_out values for better learning.I also done some experiment with Vocab length and fine tuned with following 
parameter
MAX_VOCAB_LENGTH =200 
MAX_VOCAB_LENGTH_CODE =300 

INPUT_DIM = len(SRC.vocab)
OUTPUT_DIM = len(TRG.vocab)
HID_DIM = 256
ENC_LAYERS = 3
#DEC_LAYERS = 2
DEC_LAYERS = 3
ENC_HEADS = 8
DEC_HEADS = 8
ENC_PF_DIM = 1024
DEC_PF_DIM = 1024
ENC_DROPOUT = 0.2
DEC_DROPOUT = 0.2

## Training and scheduling:
For finer tuning use the ReduceLROnPlateau scheduler with low learning rate. It helps in further fine tuning  the model's perplexity.
scheduler = ReduceLROnPlateau(optimizer, patience=5, min_lr=1e-8,verbose=True)

##Metrics used Perplexity and the model is able to achieve the following result for the assignment dataset. 
Epoch: 02 | Time: 0m 3s
Train Loss: 0.749 | Train PPL:   2.114
Val. Loss: 0.696 |  Val. PPL:   2.005

## Display graph for attention has been captured for generated input.
## Loss function: I settled with crossentropy loss function only as it has given me the best result.

## Sample O/p
#########################################################################################################
Description: write a python function to that performs as relu
Source Code:


 def ReLU(num ): 
     if num > 0 : 
         return num 
     return 0


Generated Code by Model:
def <unk> ): 
     if num < 0 : 
         return 0 
     return 0 
#########################################################################################################
Description: 43 write a program to convert kilometers to miles
Source Code:

 kilometers = float(input('How many kilometers ? : ' ) ) 
 conv_fac = 0.621371 
 miles = kilometers * conv_fac 
 print('%0.3f kilometers is equal to % 0.3f miles ' % ( kilometers , miles ) )


Generated Code by Model:

 kilometers = <unk> 
 conv_fac = 0.621371 
 <unk> = kilometers * conv_fac 
 <unk> kilometers is <unk> to % ( kilometers , <unk> ) ) 
#########################################################################################################
Description: 70 write a program to display the powers of 2 using anonymous function
Source Code:

 terms = 10 
 result = list(map(lambda x : 2**x , range(terms ) ) ) 

 print("The total terms are : " , terms ) 
 for i in range(terms ): 
     print("2 raised to power " , i , " is " , result[i ] )


Generated Code by Model:

 terms=10 
 result = list(map(lambda x : 2 * * x , range(terms ) ) ) 
 print("The total <unk> are : " , <unk> ) 
 for i in range(terms ): 
     print("2 raised to power " , i , result[i ] ) 
#########################################################################################################
Description: write a python function that accepts a dictionary that has unique values and returns its inversion
Source Code:
def invert_dict(input_dict ): 
     my_inverted_dict = { value : key for key , value in input_dict.items ( ) } 
     return my_inverted_dict


Generated Code by Model:
def <unk> ): 
     my_inverted_dict = { value : key for key , value in input_dict.items ( ) } 
     return my_inverted_dict 
#########################################################################################################
Description: write a function , which will find all such numbers between 1000 to 9999 that each digit of the number is an even number .
Source Code:

 values= [ ] 
 for i in range(1000 , 9999 ): 
     s = str(i ) 
     if ( int(s[0 ] ) % 2 = = 0 ) and ( int(s[1 ] ) % 2 = = 0 ) and ( int(s[2 ] ) % 2 = = 0 ) and ( int(s[3 ] ) % 2 = = 0 ): 
         values.append(s )


Generated Code by Model:

 list1 = [ 11 , -21 , 0 , 45 , 66 , -93 ] 

 for num in list1 : 

     if num % 2 = = 0 : 
         print(num , end= " ) 
#########################################################################################################
Description: write a function to return the perimeter of a isoscales triangle
Source Code:
def cal_perimeter_iso_triangle(s1 , s2 ): 
     return 2 * s1 + s2


Generated Code by Model:
def cal_perimeter_iso_triangle(s1 , s2 ): 
     return 2 * s1 + s2 
#########################################################################################################
Description: write a python program to flatten   a multidimensional list
Source Code:

 my_list=[[10 , 20 , 30 ] , [ 40 , 50 , 60 ] , [ 70 , 80 , 90 ] ] 

 flattened=[x for temp in my_list for x in temp ] 
 print(flattened )


Generated Code by Model:

 <unk> = [ 1 , 2 , 3 ] 
 <unk> = [ [ [ 4 , 5 , 6 ] , [ 7 , 8 , 9 ] ] 
 <unk> = [ ] 
 for i in <unk> ) ) ) ) ] 
#########################################################################################################
Description: write a python program which takes input a number n and print first n elements of fibonacci series
Source Code:

 N = int(input("Please enter a number " ) ) 
 first = 0 
 second = 1 
 print(first ) 
 print(second ) 
 for a in range(1 , N - 1 ): 
     third = first + second 
     print(third ) 
     first , second = second , third


Generated Code by Model:

 n = int(input("Enter number of rows : " ) ) 
 sum = 0 
 for i in range(n , 0 ): 
     sum + = <unk> ] 
     while ( i < = n ): 
         sum + = <unk> ] 
     return sum + <unk> 
#########################################################################################################
Description: write a python function to return words in a sentence in sorted order
Source Code:
def get_sorted_words(sentence ): 
     words = [ word for word in sentence.split ( ) ] 
     words.sort ( ) 
     return words


Generated Code by Model:
def <unk> ): 
     words = [ word for word in sentence.split ( ) ] 
     words.sort ( ) 
     return words 
#########################################################################################################
Description: write a function to return the area of a right angle triangle
Source Code:

 def cal_rt_triangle_area(base : float , height : float ) - > float : 
     if base and height : 
         return ( base * height ) / 2 
     else : 
         return None


Generated Code by Model:
def <unk> : float , height : float ) - > float : 
     if base and height : 
         return ( base * height ) / 2 
     else : 
         return None 
#########################################################################################################
Description: write a python function to count set bits in a number
Source Code:


 def count_set_bits(n ): 
     count = 0 
     while n : 
         n & = n - 1 
         count + = 1 
     return count


Generated Code by Model:
def <unk> ): 
     count = 0 
     while n : 
         count + = 1 
         n & = 1 
         n - 1 
         n = 1 
     return count 
#########################################################################################################
Description: generate a list by list comprehension
Source Code:
list=[x for x in range(10 ) ] 
 print(f"List Generated by list comprehension:{list } " )


Generated Code by Model:
list = [ x for x in range(10 ) ] 
 <unk> by list <unk> } " ) 
#########################################################################################################
Description: write a python program to find and print volume of a sphere for which diameter d is given
Source Code:
import math 

 diameter=12 . 
 radius = diameter/2 .


Generated Code by Model:
import math 
 radius = 10 
 print(f'Area : { 2 * math.pi * radius } ' ) 
#########################################################################################################
Description: 30 . write a python function to find hcf or gcd and return the value
Source Code:
def compute_hcf(x , y ): 
     if x > y : 
         smaller = y 
     else : 
         smaller = x 
     for i in range(1 , smaller+1 ): 
         if((x % i = = 0 ) and ( y % i = = 0 ) ): 
             hcf = i 
     return hcf


Generated Code by Model:
def compute_hcf(x , y ): 
     while(y ): 
         x , y = y , x % y 
     return x 
#########################################################################################################
Description: 16 how to add list numbers in python
Source Code:

 numbers = [ 1 , 2 , 3 , 4 , 5 , 1 , 4 , 5 ] 
 Sum = sum(numbers )


Generated Code by Model:

 list1 = [ 11 , -21 , 0 , 45 , 66 , -93 ] 

 for num in list1 : 

     if num > = 0 : 
         print(num , end= " " ) 
#########################################################################################################
Description: write a function to calculate the temprature t of ideal gas based on ideal gas equation pressure p and volume v given
Source Code:


 def find_temp_of_ideal_gas(pressure : float , volume : float , n : float ) - > float : 
     r = 8.3145 
     return ( pressure * volume ) / n * r


Generated Code by Model:


 def <unk> : float , volume : float , n : float ) - > float : 
     r = 8.3145 
     return ( pressure * volume ) / n * r 
#########################################################################################################
Description: write a python function to return octal value of a given integer
Source Code:
def int_to_oct(a ): 
     return oct(a )


Generated Code by Model:
def int_to_oct(a ): 
     return oct(a ) 
#########################################################################################################
Description: write a program to join two lists
Source Code:

 list1 = [ " a " , " b " , " c " ] 
 list2 = [ 1 , 2 , 3 ] 

 list3 = list1 + list2 
 print(list3 )


Generated Code by Model:

 list1 = [ 1 , 2 , 3 ] 
 list2 = [ 5 , 6 ] 
 final = [ a + b for ( a , b ) in list2 ) ] 
 <unk> of every pair of numbers from two lists:{final } " ) 
#########################################################################################################
#########################################################################################################
Description: write a function that sorts list of numbers and returns top element
Source Code:


 def biggest_no(l : list ) - > int : 
     sorted(l )


Generated Code by Model:
def <unk> : list ) - > int : 
     sorted(l ) 
#########################################################################################################
Description: write a python function to return the factorial of a number
Source Code:
def fact(n ): 
     if n = = 1 : 
         return n 
     else : 
         return n * fact(n - 1 )


Generated Code by Model:


 def factorial(n ): 
     if n = = 1 : 
         return 1 
     else : 
         return n * factorial(n - 1 ) 
#########################################################################################################
Description: write a function to calculate the moment of inertia of a ring of mass m and radius r
Source Code:
def cal_mi_ring(mass : float , radius : float ) - > float : 
     return mass * ( radius**2 )


Generated Code by Model:
def cal_mi_ring(mass : float , radius : float ) - > float : 
     return mass * ( radius**2 ) 
#########################################################################################################
Description: write a python function that returns true if the sum of two provided numbers is even
Source Code:
def is_prod_even(num1 , num2 ): 
     sum = num1 + num2 
     return not sum % 2


Generated Code by Model:
def <unk> , num2 ): 
     sum = num1 + num2 
     return not sum % 2 
#########################################################################################################
Description: write a python function that returns a dictionary with the area and perimeter of a rectangle
Source Code:
def calculate_rect_properties(width , height ): 
     return { ' perimeter ' : 2 * ( width + height ) , ' area ' : width * height }


Generated Code by Model:
def <unk> , height ): 
     return { ' <unk> ' : 2 * ( width + height ) , ' area ' : width * height } 
#########################################################################################################
Description: write a function that returns uppercase string
Source Code:
def str_uppercase(s : str ): 
     return s.upper ( )


Generated Code by Model:
def <unk> : str ): 
     return s.upper ( ) 
#########################################################################################################
Description: write a function that returns gets the derivative of exponential of x
Source Code:
def derivative_exp(x : float ) - > float : 
     import math 
     return math.exp(x )


Generated Code by Model:
def derivative_exp(x : float ) - > float : 
     import math 
     return math.exp(x ) 
#########################################################################################################
Description: given a python list . turn every item of a list into its square
Source Code:
aList = [ 1 , 2 , 3 , 4 , 5 , 6 , 7 ] 
 aList = [ x * x for x in aList ] 
 print(aList )


Generated Code by Model:
aList = [ 1 , 2 , 3 , 4 , 5 , 6 , 7 , 8 ] 
 aList = [ x * x for x in aList ] 
 print(aList ) 
#########################################################################################################
Description: write a python function to get the cumulative sum of a list
Source Code:
def Cumulative(lists ): 
     cu_list = [ ] 
     length = len(lists ) 
     cu_list = [ sum(lists[0 : x:1 ] ) for x in range(0 , length + 1 ) ] 
     return cu_list[1 :]


Generated Code by Model:
def Cumulative(lists ): 
     cu_list = [ ] 
     length = len(lists ) 
     cu_list = [ sum(lists[0 : x:1 ] ) for x in range(0 , length + 1 ) ] 
     return cu_list[1 :] 
#########################################################################################################
Description: write a program to swap two variables inplace
Source Code:
a , b = b , a


Generated Code by Model:
a = b , b = b , a 
#########################################################################################################
Description: write a python function that takes an integer as input and returns the factorial of that number
Source Code:

 def factorial(n ): 
     return 1 if ( n = = 1 or n = = 0 ) else n * factorial(n - 1 )


Generated Code by Model:


 def factorial(n ): 
     if n = = 1 : 
         return 1 
     else : 
         return n * factorial(n - 1 ) 
#########################################################################################################
Description: write a function to return the total surface area of a cuboid of length l , bredth b and height h
Source Code:
def cal_surface_area_cuboid(l , b , h ): 
     return 2 * ( l * b + b * h + h * l )


Generated Code by Model:
def cal_surface_area_cuboid(l , b , h ): 
     return 2 * ( l * b + b * h + h * h + h 
#########################################################################################################
Description: removes the item at the given index from the list and returns the removed item
Source Code:
my_list1=[4 , 3 , 2 , 9 , 10 , 44 , 1 , 9 , 12 ] 
 index=4 
 print(f"Sum of two list:,{my_list1.pop(index ) } " )


Generated Code by Model:
my_list1 = [ 4 , 3 , 2 , 9 , 10 , 44 , 1 ] 
 index = 9 , 12 , 12 ] 
 <unk> of two <unk> } " ) 
#########################################################################################################
Description: write a python function that returns the input list sorted in descending order
Source Code:

 def sort_descending(list_to_be_sorted ): 
     return sorted(list_to_be_sorted , reverse = True )


Generated Code by Model:
def sort_descending(list_to_be_sorted ): 
     return sorted(list_to_be_sorted , reverse = True ) 
#########################################################################################################
Description: write a python function to compute 5/0 using try except
Source Code:
try : 
     print("Division = { } " .format(5 / 0 ) ) 
 except ZeroDivisionError : 
     print("5 can not be divided by O " )


Generated Code by Model:

 <unk> = { } 
 for i in range(1 , 11 ): 
     <unk> = i , <unk> ) if ( i % 3 = = 0 ): 
         <unk> = i 
 <unk> )
#########################################################################################################
Description: write a python program to find the square root
Source Code:

 num = 8 
 num_sqrt = num**0.5 
 print('The square root of % 0.3f is % 0.3f ' % ( num , num_sqrt ) )


Generated Code by Model:

 num = int(input("Enter a number : " ) ) 
 if num > 0 : 
     print("Positive number " ) 
 elif num = = 0 : 
     print("Zero " ) 
 else : 
     print("Negative number " ) 
#########################################################################################################
Description: write a python program to print the character of an ascii value
Source Code:
value = 65 
 print(f'The ASCII value { value } is of the character { chr(value ) } ' )


Generated Code by Model:
<unk> ' 
 print("The ASCII value of ' " + c + " ' is " , ord(c ) ) 
#########################################################################################################
Description: write a python function to prepend a single value in front of an iterator
Source Code:
def prepend(value , iterator ): 
     import itertools 
     return itertools.chain([value ] , iterator )


Generated Code by Model:
def <unk> , <unk> ): 
     import itertools 
     return <unk> ) 
#########################################################################################################
Description: write a python function to calculate the dot product of two lists
Source Code:
def dot(l1 , l2 ): 
     return sum(x * y for x , y in zip(l1 , l2 ) )


Generated Code by Model:
def add_two_lists(list1 , list2 ): 
     list1 = [ 1 , 2 , 3 ] 
     list2 = [ 4 , 5 , 6 ] 
     sum_list = [ ] 
     for ( item1 , item2 ) in zip(list1 , list2 ): 
         sum_list.append(item1 + item2 ) 
     return sum_list 
##########################################################################################################
Description: write a python function to create multiplication table from the user provided number
Source Code:


 def multiplication_table(n ): 
     for i in range(1 , 11 ): 
         print(n , ' x ' , i , ' = ' , n * i )


Generated Code by Model:


 def <unk> ): 
     for i in range(1 , 11 ): 
         print(n , ' x ' , ' , ' , ' y ' , ' , i , ' = ' , i ) 
#########################################################################################################
Description: write a python function to get the cumulative sum of a list
Source Code:
def Cumulative(lists ): 
     cu_list = [ ] 
     length = len(lists ) 
     cu_list = [ sum(lists[0 : x:1 ] ) for x in range(0 , length + 1 ) ] 
     return cu_list[1 :]


Generated Code by Model:
def Cumulative(lists ): 
     cu_list = [ ] 
     length = len(lists ) 
     cu_list = [ sum(lists[0 : x:1 ] ) for x in range(0 , length + 1 ) ] 
     return cu_list[1 :] 
#########################################################################################################
Description: write a python program to remove words less than a specified length from a sentence
Source Code:
sentence = ' this is my sentence and i will write it my way ' 
 minlength = 3 
 result = [ word for word in sentence.split ( ' ' ) if len(word ) > = minlength ]


Generated Code by Model:
sentence = ' this is my sentence and i will write it my way ' 
 minlength = 3 
 result = [ word for word in sentence.split ( ' ' ) if len(word ) < = minlength ] 
#########################################################################################################
Description: write a python program to print equal length of string
Source Code:

 print('ab'.zfill(5 ) )


Generated Code by Model:

 test_list = [ ( 4 , 5 ) , ( 3 , 6 , 7 ) , ( 8 , 6 , 7 ) ] 

 K = 7 
 res = [ ele for ele in test_list if len(ele ) ! = K ] 

 print("Filtered list : " + str(res ) ) 
#########################################################################################################
Description: write a python function to get the surface_area of a cylinder with radius & height as input
Source Code:
def cylinder_surface_area(radius , height ): 
     surface_area=3.14 * ( radius * * 2 ) + ( 2 * 3.14 * radius * height ) 
     return surface_area


Generated Code by Model:
def <unk> , radius ): 
     pi = 3.14 
     return pi * ( radius**2 ) * height 
#########################################################################################################
Description: write a program to convert date to timestamp and print the result
Source Code:
import time 
 import datetime 
 str1 = " 20/01/2020 " 
 element = datetime.datetime.strptime(str1 , " % d/%m/%Y " ) 
 timestamp = datetime.datetime.timestamp(element ) 
 print(timestamp )


Generated Code by Model:
import datetime 
 given_date = datetime(2020 , 7 , 26 ) 
 print(given_date.strftime('%A ' ) 
#########################################################################################################
Description: write a python program to combine two dictionary by adding values for common keys
Source Code:
dict1 = { ' a ' : 12 , ' for ' : 25 , ' c ' : 9 } 
 dict2 = { ' Geeks ' : 100 , ' geek ' : 200 , ' for ' : 300 } 
 for key in dict2 : 
     if key in dict1 : 
         dict2[key ] = dict2[key ] + dict1[key ] 
     else : 
         pass


Generated Code by Model:
dict1 = { ' a ' : 12 , ' for ' : 25 , ' c ' : 9 } 
 dict2 = { ' Geeks ' : 100 , ' geek ' : 200 , ' for ' : 300 } 
 for key in dict2 : 
     if key in dict1 : 
         dict2[key ] = dict2[key ] + dict1[key ] 
     else : 
         pass 
#########################################################################################################
Description: python function that takes a positive integer and returns the sum of the cube of all the positive integers smaller than the specified number .
Source Code:
def sum_of_cubes(n ): 
     n -= 1 
     total=0 
     while n > 0 : 
         total + = n * n * n 
         n -= 1 
     return total 



 print("Sum of cubes : " , sum_of_cubes(3 ) )


Generated Code by Model:
def f(n ): 
     if n = = 0 : 
         return 0 
     else : 
         return f(n - 1 ) + f(n - 2 ) + 100 
#########################################################################################################
Description: write a function that inverts a dictionary with non - unique values . keys that map to the same values should be appended to a list in the inverted dictionary
Source Code:
def invert_dict_non_unique(my_dict ): 
     my_inverted_dict = dict ( ) 
     for key , value in my_dict.items ( ): 
         my_inverted_dict.setdefault(value , list()).append(key ) 
     return my_inverted_dict


Generated Code by Model:
def invert_dict_non_unique(my_dict ): 
     my_inverted_dict = dict ( ) 
     for key , value in my_dict.items ( ): 
         my_inverted_dict.setdefault(value , list()).append(key ) 
     return my_inverted_dict 
#########################################################################################################
Description: write a function to check a valid email i d
Source Code:

 import re 


 def check(email ): 
     regex = ' ^[a - z0 - 9]+[\._]?[a - z0 - 9]+[@]\w+[.]\w{2,3}$ ' 
     if ( re.search(regex , email ) ): 
         print("Valid Email " ) 
     else : 
         print("Invalid Email " )


Generated Code by Model:

 import re 

 regex = ' ^[a - z0 - 9]+[\._]?[a - 9]+[@]\w+[.]\w{2,3}$ ' 


 def check(email ): 
     if ( re.search(regex , email ) ): 
         print("Valid Email " ) 
     else : 
         print("Invalid Email " ) 
#########################################################################################################
Description: write a program which accepts a sequence of words separated by whitespace as input to print the words composed of digits only .
Source Code:
import re 
 s = raw_input ( ) 
 print(re.findall("\d+ " , s ) )


Generated Code by Model:
import re 
 s = input ( ) 
 print(re.findall("\d+ " , s ) ) 
#########################################################################################################
Description: write a python program to swap the first and last value of a list
Source Code:

 a = [ 2 , 3 , 8 , 9 , 2 , 4 , 6 ] 
 n = len(a ) 
 temp = a[0 ] 
 a[0 ] = a[n - 1 ] 
 a[n - 1 ] = temp 
 print("New list is : " ) 
 print(a )


Generated Code by Model:

 a = [ 1 , 2 , 3 , 4 , 5 ] 
 b = [ 5 , 6 , 7 , 8 , 9 ] 
 a = b 
 print(a ) 
