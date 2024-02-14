---
date: "2021-04-21"
tags: ["pty", "cli", "tutorial"]
title: "Understanding and Creating Pseudo-Terminals (PTY)"
---

> This content was written based on Chapter 6 of the [**Hands-On System Programming with Go**](https://www.packtpub.com/product/hands-on-systems-programming-with-go/9781789804072) book written by Alex Guerrieri. 

## Introduction 

Pseudo-terminals, or pseudo-teletypes, are applications that run under a terminal. Many programs are built as pseudo-terminal because it enables interactive use from inside a terminal without needing any graphical interface.

As a software engineer, we use many applications that are built as pseudo-terminal. One of them is the SSH Client. SSH means Secure Shell, and the shell is a computer program that exposes an operating system's services generally as a command-line interface. 

**PTY** are the initials of pseudo-teletype and is the formal way to call a pseudo-terminal. This name is inherited from TTY, which means teletype, and is the name for typewriters. Typewriters are electromechanical systems connected to a computer, capable of sending information to the device to print it. TTY is also the name of a special device used by deaf or speech-impaired people to communicate through the telephone. In this device, typed messages go back and forth instead of talking and listening to each other. In this case, a TTY is required at both ends. 

In order to be considered a PTY, an application needs to be capable of:
- accepting input from the user;
- sending input to the console and receiving back the output;
- showing the received output to the user.

## Creating a Basic PTY

A PTY is composed of three main parts:
- input manager;
- command selector;
- command execution.

We will go through implementing these three major features of a PTY, starting with the **Input Manager**.

### Input Manager

We will take baby steps and start simple. Let's create a simple [go module](https://blog.golang.org/using-go-modules) project with an empty `main.go` file.

We will write a program with a for loop that will read user inputs through out the command line standard input. In Golang, we wil achieve that by listening to the os.Stdin and scanning the inputs with bufio.Scanner. The code should look similar to the following: 

```golang
func main() {
	s := bufio.NewScanner(os.Stdin) // read input
	w := os.Stdout // write output
	fmt.Fprintln(w, "Welcome to my PTY! Type something and I will echo it...")
	for {
		s.Scan() // Get next token
		input := string(s.Bytes())
		fmt.Fprintf(w, "You wrote %q\n", input) 
	}
}
```

Right now, the only way to stop the execution of the application is killing it with a kill command. We are interested to allow the uset to input a command that will terminate the application execution. We can achieve that with a small change to the previous code: 

```golang
func main() {
	s := bufio.NewScanner(os.Stdin)
	w := os.Stdout
	fmt.Fprintln(w, "Welcome to my PTY! Type something and I will echo it...")
	for {
		s.Scan() // get next token
		input := string(s.Bytes())
		if input == "exit" {
			return
		}
		fmt.Fprintf(w, "You wrote %q\n", input)
	}
}
```

Now the application has an exit point other than the kill command. For now, it does not implement any command besides the exit one, and all it does is print back whatever is typed by the user.


### Command Selector

To add new commands to our PTY, we need to be able to interpret different commands correctly. Commands might accept arguments, so we start by splitting the input into those arguments. We can use the `strings.Split` function to do the trick. 

```golang
args := strings.Split(input, " ")
cmd := args[0]
args = args[1:]
```
The split function will break the input at the separator we provide, which will be a single space `" "`. This is the same logic that the operating system applies to arguments passed to a process.   

Breaking the input into `command` and `arguments`, make it possible to execute any sort of check on the `cmd` value. In this case, we will create a switch statement that will execute a specific function for a given command. 

```golang
switch cmd {
case "exit": 
    return
case "someCommand":
    someCommand(w, args) 
case "anotherCommand":
    anotherCommand(w, args) 
}
```

This allows adding any command by defining a function and adding a new case to the switch.

### Command Execution

For the command execution, we can define how a command function should look alike by creating a variable out of the function signature.

```golang
var cmdFunc func(w io.Writer, args []string) (exit bool)
```

With that, we can write the function for the `exit` command.

```golang
func exitCmd(w io.Writer, args []string) bool {
	fmt.Fprintln(w, "Bye! :D")
	return true
}
```

Now, we update our main code to make use of the new `cmdFunc` and `exitCmd`.

```golang
switch cmd {
case "exit":
    cmdFunc = exitCmd
}
if cmdFunc == nil {
    fmt.Fprintln(w, "command %q not found", cmd)
    continue
}
if cmdFunc(w, args) {
    return
}
```

With that, we can play around and implement any command function of the same type as `cmdFunc`. Let’s create one more command as an example, a `shuffle` command, which will print the arguments in a shuffled order. To do that, we will use the `match/rand` package.

```golang
func shuffleCommand(w io.Writer, args []string) bool {
	rand.Seed(time.Now().UnixNano())
	rand.Shuffle(len(args), func(i, j int) {
		args[i], args[j] = args[j], args[i]
	})
	for i := range args {
		if i > 0 {
			fmt.Fprintf(w, " ")
		}
		fmt.Fprintf(w, "%s", args[i])
	}
	fmt.Fprintln(w)
	return false
}
```

Now we add it to the main swith: 

```golang
switch cmd {
case "exit":
    cmdFunc = exitCmd
case "shuffle":
    cmdFunc = shuffleCommand
}
```

That's it, you've just created a PTY with two commands `exit` and `shuffle`.

## Conclusion

With that, we learned that creating a basic PTY is pretty simple and straightforward. Our PTY has a huge room to grow and I encourage you to try to improve it. Some possible improvements are:
- Create a help command and add descriptions to each of the commands
- Suggest commands using the `levenshtein algorithm`
- Make commands extensible without needing to add new commands to the main package

The list of improvements is infinite and I can stay here thinking and writing the list forever. 

Thanks for reading until the very end and, if you interested to see the code I wrote for this article, [here is the go playground link](https://play.golang.org/p/HncuuNtfSmO). The playground doesn't have a console to input text, so the PTY won't work there and you need to build it in your machine to try it out.