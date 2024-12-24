
php starting and ending tag   <?php  .....      ?>


to print something we use echo 
```
echo "Nova !"
```


to comment we use // and /*
```
// sample comment    
/* multiline */
```


variable - $name  and { $name}-echo

escape char \$
echo true - output 1 ,false - output nothing

operator precedence  --  ( )    **  * / %    + -



$_GET  and $_POST
$_GET -       $_GET["username"]
			data appended to the url
			 not secure
			 char limit
			 bookmark is possible
			 get requests can be cached
$_POST -      $_POST
			 Data is packed in http request
			 more secure
			 no data limit
			 cannot bookmark
			 get requests are cahed
			 better for submitting creds




Logical operators

&&              True if both are true
||                   True if atleast one is true
!                     True if false



Jump statements

if
```
if(condition){ .. }elseif{.....}else{.....}
```


switch
```
switch(condition){
	case "1":
		...
	case "2":
		...
		...
	default:
}
```


for loops

```
for(init;condi;incre){   ....   }
```


while
```
while(condition){ ...}
```




Arrays
```
$variable = array("1","2","3");
$variable = array("1"=> 1,"2"=> 2,"3"=> 3);
```

we cant directly print the whole array; and we can use [ ] to access elements foreach to access all elements
```
foreach($variable as $var){
..
}

foreach($variable as $key => $value){

}
```

 array_push($var, value)  --   to add element to end of list
 array_pop($var) -- del last elemnt
 array_shify($var) -- moves element one step front
array_reverse($var)


isset()  - if variable is declared
empty()   if variable is not declared





functions

```
function name(){   }
```



string functions

strtolower() - str
strtoupper
trim(var,s)
str_pad(var,20,"0")
str_replace(from , to)
strcmp()
strlen() 
strpos()