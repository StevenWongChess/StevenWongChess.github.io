---
layout: post
title:  "Notes for OCaml From the Very Beginning"
date:   2023-10-04
categories: Ocaml
---

#### Notes for `OCaml From the Very Beginning`

Link to the book: https://johnwhitington.net/ocamlfromtheverybeginning/mlbook.pdf

My Take-away: This book is all about very very basic grammer and have good example



###### Logic Flow

```ocaml
(* if, else can be else () or omitted *)
if ... then ... else ...
(* enforce imperative programming, 
otherwise need to use if to make life hard *)
begin
	...
	...
end
(* for loop *)
for x = 1 to 5 do ... done
(* while loop *)
while ... do ... done
```



###### Func Def & Pattern Match

```ocaml
(* recursive function,
use pattern match over if else *)
let rec factorial a = match a with
		0 -> 1
  | 1 -> 1
  | _ -> a * factorial (a - 1)
(* put together 0 | 1 *)
let rec factorial a = match a with
		0 | 1 -> 1
  | _ -> a * factorial (a - 1)
```



###### List Def & Funs

```ocaml
(* MUST have same type *)
[1; 2; 3]
(* Notice time complexity *)
1 :: [2, 3] (* :: is called cons *)
[1,2] @ [3]
(* destruction for list*)
let rec sum l = 
	match l with
		[] -> 0 
	| _::t -> h + sum t (* can also be h::t*)
(* Recursive functions which do not build up a growing intermediate expression are known as tail recursive *)
(* optimize: no extra space*)
let rec length_inner l n = 
	match l with
    [] -> n
  | h::t -> length_inner t (n + 1)
(* append *)
let rec append a b = 
	match a with
    [] -> b
  | h::t -> h :: append t b
(* reverse *)
let rec rev l = 
	match l with
    [] -> []
  | h::t -> rev t @ [h]
  
(* 
	NEED to deal with incomplete pattern match in future
	i.e. n is larger than the size of list
*)  
(* 1. take *)
let rec take n l =
	if n = 0 then [] else
		match l with
			h::t -> h :: take (n - 1) t
(* 2. drop *)
let rec drop n l =
	if n = 0 then l else
		match l with
			h::t -> drop (n - 1) t

(* insertion sort *)
let rec insert x l = 
	match l with
    [] -> [x]
  | h::t ->
		if x <= h
			then x :: h :: t 
			else h :: insert x t
let rec sort l = 
	match l with 
		[] -> []
  | h::t -> insert h (sort t)
  
(* merge sort*)
let rec merge x y = 
	match x, y with
    [], l -> l
  | l, [] -> l
  | hx::tx, hy::ty ->
		if hx < hy
			then hx :: merge tx (hy :: ty) 
			else hy :: merge (hx :: tx) ty
let rec msort l = 
	match l with
     [] -> []
   | [x] -> [x]
   | _ ->
			let left = take (length l / 2) l in 
				let right = drop (length l / 2) l in
           merge (msort left) (msort right)
                     
(* map *)
let rec map f l = 
	match l with
		[] -> [] 
	| h::t -> f h :: map f t
	
(* anonymous function *)
let evens l =
	map (fun x -> x mod 2 = 0) l
	
(* even abbr for function *)
( <= )
( <= ) 4 5
```



###### Include File

- Write code in `list.ml`
- Use `#use "list.ml";;`



###### Run-time Exception

```ocaml
exception Problem;;
raise Problem (* to trigger *)

(* handle instead of raise*)
let safe_divide x y = 
	try x / y with
		Division_by_zero -> 0
(* More on page 57 *)
```



###### Dict & Pair

```ocaml
let p = (1, '1');;
(* similar to first, second in C++ *)
let fst p = match p with (x, _) -> x 
let snd p = match p with (_, y) -> y
(* in short *)
let fst (x, _) = x 
let snd (_, y) = y

(* lookup *)
let rec lookup x l = 
	match l with
		[] -> raise Not_found 
	| (k, v)::t ->
			if k = x then v else lookup x t
			
(* add *)
let rec add k v d = 
	match d with
    [] -> [(k, v)]
  | (k', v')::t ->
			if k = k'
				then (k, v) :: t
				else (k', v') :: add k v t
```



###### Trick: Partial Application

Giving fewer than the full number of arguments

```ocaml
let add x y = x + y
let f = add 6
```



###### Define Own Type

```ocaml
type colour = Red | Green | Blue | Yellow;;

(* usage of "of" *)
type colour = Red
| Green
| Blue
| Yellow
| RGB of int × int × int

(* option *)
type 'a option = None | Some of 'a;;

(* rec type *)
type 'a sequence = Nil | Cons of 'a * 'a sequence;;

(* high level idea: often the functions are easy to write once we have decided on appropriate type, example calculator*)
```



###### Tree & BST

Focus on the example and functions there, easy there



###### I/O

```ocaml
print_int 100;; (* will return unit, which means nothing *)
print_string
print_newline

(* 
	multi Output, 
	";" throws away the result, which will normally be unit anyway 
	no ";" at the end because need to return unit
*)
print_int k; print_newline (); print_string v; print_newline ()

(* Input *)
let rec read_dict () = 
	try
		let i = read_int () in 
			if i = 0 then [] else
				let name = read_line () in 
					(i, name) :: read_dict ()
	with
    Failure _ ->
      print_string "This is not a valid integer. Please try again.";
      print_newline ();
      read_dict ()
      
(* 
	File IO 
	2 types: in_channel, out_channel
	see detail example on page 94
	End_of_file exception to stop
*)
let ch = open_out/open_in filename in
	output_string ch xxx;
	output_char ch xxx;
	close_out/close_in ch
usually need to transfer int to string via string_of_int
```



###### Reference

```ocaml
let x = ref 0;;
let p = !x;; (* dereference *)
x := 50;; (* update *)

(* Very detailed word_count example on page 102 *)
```



###### Array

```ocaml
let a = [|1; 2; 3; 4; 5|];;
a.(0);; (* a[0] *)
a.(4) <- 100;; (* a[4] = 100 *)
Array.make 6 true;; (* vector<bool> v(6, true) *)
Array.length a;; (* a.size() *)
```



###### Float

Need to add . after operator for float



###### Standard Library

List, Array, Char, String, Random, Buffer, Printf



