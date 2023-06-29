# FortranCompiler
## Idea of splitting :

The line tokens = re.findall('\w+|\W', text) uses regular expressions to split the input text into a list of tokens. Here's how it works:

\w+ matches one or more word characters (letters, digits, or underscores).
\W matches any non-word character.
| is the OR operator in regular expressions, so \w+|\W matches either one or more word characters or any non-word character.
By using re.findall(pattern, text), the function finds all occurrences of the pattern in the text and returns them as a list of strings. Each string represents a token in the input text.

For example, given the input text "program my_program integer x = 10;", the resulting list of tokens would be ['program', 'my_program', 'integer', 'x', '=', '10', ';'].

The purpose of splitting the text into tokens is to analyze and classify each token according to its type, as defined by the Token_type enum. This classification helps in further processing or understanding the structure and meaning of the program code





Some special parts in scanner code :
Handling comments by ignoring what is after! :
In following lines of code:
Please note as Fortan keep appending string to comment section until newline is entered that why \n condition was put and at the same time we are ignoring the spaces as not to count as token type.




![pic1](https://github.com/Zeinahesham308/FortranCompiler/assets/108671897/ec92bba7-22d3-4c33-bab6-06171211b929)




handling some reserverd words even if they are entered with space :
![pic2](https://github.com/Zeinahesham308/FortranCompiler/assets/108671897/dfd6ad6d-8f23-49f9-93d1-efa40a808af9)


Part to take string as a whole if it was entered right between double quotations or single quotation as in fortran you can write string between “” or ‘’ code for this part :


![pic3](https://github.com/Zeinahesham308/FortranCompiler/assets/108671897/3276ccd0-6fac-4a6b-9188-10a5d5fdabea)


Handling .true. and .false. and cases where . was not put at the end 



![pic4](https://github.com/Zeinahesham308/FortranCompiler/assets/108671897/26fdae1c-fc7a-481f-9c30-06708a49357b)


## Parser 
### Grammar Rules :

PROGRAM Header Endlines DeclSec Statements End
Header  program identifier
Declsec  Implicit VarDecls | ε
Implicit -> implicit none EndLines
End  end program identifier 

Endlines -> Endlines endline | endline
{
#Left Recursion 
Endlines-> endline Elines 
ELines-> endline Elines | ε
}






VarDecls → VarDecls VarDecl | VarDecl 
{
                #Left Recursion 
VarDecls VarDecl VarDeclsDash
VarDeclsDash VarDecl VarDeclsDash | ε
}

IdList → IdList , identifier | identifier
{
           #Left Recursion 
IdList->  identifier IdListDash
IdListDash-> , identifier IdListDash  |  ε
}

MoreDecl , identifier = Expression MoreDecl |  ε
Number  integerno|realno
 Bool  .true. |.false.



VarDecl →
 Datatype ::  IdList EndLines
| Datatype , parameter ::  IdList  = Number EndLines
| Datatype ::  IdList = AssignmentDash MoreDecl EndLines
| ε
{
#Left Factoring 
VarDecl  Datatype VarDeclDash | ε
VarDeclDash 
   :: IdList EndLines
| :: IdList = AssignmentDash MoreDecl EndLines 
|  , parameter ::  IdList  = Number EndLines

#Left Factoring 
    {
                     VarDeclDash -> 
                         :: IdList VarDeclDoubleDash EndLines 
 | , parameter ::  IdList  = Number EndLines 

                         VarDeclDoubleDash-> = AssignmentDash MoreDecl| ε 
}
}
Datatype → integer | real | complex | logical| Character
Character-> character|character( len = constant )
{
          #Left Factoring 
Character->character CharacterDash
CharacterDash->   ( len = constant )  |  ε 
}


Statements →  Statements  Statement | Statement
{
        #Left Recursion 
Statements Statement  StatementsDash
StatementsDash Statement  StatementsDash |  ε
}

Statement → 
 read*, IDlist  EndLines
| Assignment EndLines
         | Printt EndLines
         | IF EndLines
         | do DoParams EndLines Statements enddo EndLines


DoParams  identifier = DoindentifierList | ε

Assignment
 identifier = Expression
| identifier = Bool
| identifier = string 
|"string"
|’string’
{
#Left Factoring
Assignment -> identifier = AssignmentDash
AssignmentDash -> Expression | Bool |"string"|’string’
}

Printt  print*  Prints  | print*  Expression 
{
#Left Factoring
Printt  print*  PrintDash
PrintDash-> Prints | Expression
}




Prints  Prints PrintDecl| PrintDecl 
{
       #Left Recursion
Prints→ PrintDecl PrintsDash
PrintsDash  PrintDecl PrintsDash | ε
}


 PrintDecl   , PrintList | ε
PrintList   identifier | ‘ string ’ | “string”


IF  if Condition then EndLines Statements Else endif 
Else Elseif| else EndLines Statements | ε
Elseif elseif Condition then EndLines Statements Else |ε





DoidentifierList 
  DoBoundaries, DoBoundaries, DoBoundaries
 | DoBoundaries, DoBoundaries
{ 
       #Left Factoring
DoidentifierList  DoBoundaries, DoBoundaries DoidentifierListDash
DoidentifierListDash ,  DoBoundaries | ε 
        }

     DoBoundaries  identifier| Integerno







	









Condition →( Expression RelOp Expression)

Expression → Expression AddOp Term | Term 
{ 
        #Left Recursion

Expression  Term ExpressionDash
ExpressionDash  AddOp Term ExpressionDash | ε

}

Term → Term MultiOp Factor | Factor
{
          #Left Recursion
Term -> Factor TermDash
TermDash-> MultiOp Factor TermDash | ε

}

Factor → identifier | Number | ( Expression )

RelOp = > | >| <= | < |== | !=
AddOp → + | -
MultOp → * | / 

### Test Cases 


program ifProg
implicit none 
if  (a < 20) then  
  print*, "a is less than 20" 
endif 
print*, "value of a is ", a 
end program ifProg


program ifElseProg
implicit none
   ! local variable declaration
   integer :: a = 100
 
   ! check the logical condition using if statement
   if (a < 20) then
   
   ! if condition is true then print the following 
   print*, "a is less than 20"
   else
   print*, "a is not less than 20"
   endif
       
   print*, "value of a is ", a
	
end program ifElseProg



program ifElseIfElseProg
implicitnone

   integer :: a = ((2+3)*(4+5))+5/7*66

end program ifElseIfElseProg
_________________________________________________________________________________________________________

program ifElseIfElseProg
implicit none

   if (a < 10)  then
  
      print*, "Value of a is 10" 
   if (b >= 200)  then
  

   ! if inner if condition is true 
   print*, "Value of a is 100 and b is 200" 
  
   endif
   endif
   print*, "exact value of a is ", a
   print*, "exact value of b is ", b

end program ifElseIfElseProg
_________________________________________________________________________________________________________

program ifElseIfElseProg
implicit none

 integer :: a = 100
 
   if (a < 10)  then
  
      print*, "Value of a is 10" 
   if (b >= 200)  then
  

   ! if inner if condition is true 
   print*, "Value of a is 100 and b is 200" 
   elseif (c>100) then
     a= 1000
    else 
          print*, "Value of " 
   endif
   endif
   print*, "exact value of a is ", a
   print*, "exact value of b is ", b

end program ifElseIfElseProg
_________________________________________________________________________________________________________
! compute factorials
program ifElseIfElseProg
implicit none
 integer :: a = 100
do n = 1, 10
   nfact = nfact * n  
   ! printing the value of n and its factorial
   print*, " n", nfact   
enddo
end program ifElseIfElseProg
_________________________________________________________________________________________________________
program factorial  
implicit none  

   ! define variables
   integer :: nfact = 1   
   integer :: n 

   
   ! compute factorials   
   do n = 1, 10      
      nfact = nfact * n 
      ! print values
      print*,  "  n  ", nfact   
   enddo 
   
end program factorial 
_________________________________________________________________________________________________________

program factorial  
implicit none  

   ! define variables
   integer :: nfact = 1   
   integer :: n  
   
   ! compute factorials   
   do n = start, stop ,step       
      nfact = nfact * n 
      ! print values
      print*,  "  n  ", nfact   
   enddo 
   
end program factorial 

_____________________________________________________________________________________________________________
program factorial  
implicit none  

character (len=5)   :: name 
logical :: x =.true.
logical :: y

end program factorial 

program factorial  
implicit none  

real , parameter ::pi =3.1625
character :: message
logical:: done
done= .true.

message = "hello world"


end program factorial


program ifElseIfElseProg
implicit none

   integer :: a = 8+9 

end program ifElseIfElseProg
___________________________________________________________________________________________________________
 

program ifElseIfElseProg
implicit none

   integer :: a = 8+5/7*66

end program ifElseIfElseProg
____________________________________________________________________________________________________________

program Fibonacci
   implicit none

    integer :: f1,f2,f3,i

    i = 1
    f1 = 0
    f2 = 1
    
    do var = start , y
        f3 = f2 + f1
        f1 = f2
        f2 = f3
        i = i + 1
        if (f1<10) then
            print* ,'(I1, A, $)', f1

        else
            print* , '(I3, A, $)', f1
        endif

    enddo
    
    print* , '...'

    end program Fibonacci

program variables
implicit none 
integer :: amount 
real :: pi 
complex :: frequency 
character :: initial 
logical :: isOkay 
end program variables
_________________________________________________________________________________________________________
program variable 
implicit none 
print*, “The value of amount (integer) is: “ 
end program variables

________________________________________________________________________________________________________
program variable 
implicit none print*, 'hellooo' 
end program variables

_________________________________________________________________________________________________________

program variable 
implicit none 
read*, x 
end program variables
_________________________________________________________________________________________________________
program variable 
implicit none 
x=2+3*7/7*5 
end program variables

_________________________________________________________________________________________________________
program variable 
implicit none 
read*, x,y,z,h 
end program variables






