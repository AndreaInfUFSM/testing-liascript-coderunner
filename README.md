<!--
author:   Andrea Charão

email:    andrea@inf.ufsm.br

version:  0.0.1

language: en

narrator: US English Female

onload
window.CodeRunner = {
    ws: undefined,
    handler: {},

    init(url) {
        this.ws = new WebSocket(url);
        const self = this
        this.ws.onopen = function () {
            self.log("connections established");
            setInterval(function() {
                self.ws.send("ping")
            }, 15000);
        }
        this.ws.onmessage = function (e) {
            // e.data contains received string.

            let data
            try {
                data = JSON.parse(e.data)
            } catch (e) {
                self.warn("received message could not be handled =>", e.data)
            }
            if (data) {
                self.handler[data.uid](data)
            }
        }
        this.ws.onclose = function () {
            self.warn("connection closed")
        }
        this.ws.onerror = function (e) {
            self.warn("an error has occurred => ", e)
        }
    },
    log(...args) {
        console.log("CodeRunner:", ...args)
    },
    warn(...args) {
        console.warn("CodeRunner:", ...args)
    },
    handle(uid, callback) {
        this.handler[uid] = callback
    },
    send(uid, message) {
        message.uid = uid
        this.ws.send(JSON.stringify(message))
    }
}

//window.CodeRunner.init("wss://coderunner.informatik.tu-freiberg.de/")
//window.CodeRunner.init("wss://testing-coderunner.andreaschwertne.repl.co/")
window.CodeRunner.init("wss://ancient-hollows-41316.herokuapp.com/")
//window.CodeRunner.init("ws://127.0.0.1:8000/")

@end


@LIA.c:       @LIA.eval(`["main.c"]`, `gcc -Wall main.c -o a.out`, `./a.out`)
@LIA.clojure: @LIA.eval(`["main.clj"]`, `none`, `clojure -M main.clj`)
@LIA.cpp:     @LIA.eval(`["main.cpp"]`, `g++ main.cpp -o a.out`, `./a.out`)
@LIA.go:      @LIA.eval(`["main.go"]`, `go build main.go`, `./main`)
@LIA.haskell: @LIA.eval(`["main.hs"]`, `ghc main.hs -o main`, `./main`)
@LIA.java:    @LIA.eval(`["@0.java"]`, `javac @0.java`, `java @0`)
@LIA.julia:   @LIA.eval(`["main.jl"]`, `none`, `julia main.jl`)
@LIA.mono:    @LIA.eval(`["main.cs"]`, `mcs main.cs`, `mono main.exe`)
@LIA.nasm:    @LIA.eval(`["main.asm"]`, `nasm -felf64 main.asm && ld main.o`, `./a.out`)
@LIA.python:  @LIA.python3
@LIA.python2: @LIA.eval(`["main.py"]`, `python2.7 -m compileall .`, `python2.7 main.pyc`)
@LIA.python3: @LIA.eval(`["main.py"]`, `none`, `python3 main.py`)
@LIA.r:       @LIA.eval(`["main.R"]`, `none`, `Rscript main.R`)
@LIA.rust:    @LIA.eval(`["main.rs"]`, `rustc main.rs`, `./main`)
@LIA.v:       @LIA.eval(`["main.v"]`, `v main.v`, `./main`)
@LIA.zig:     @LIA.eval(`["main.zig"]`, `zig build-exe ./main.zig -O ReleaseSmall`, `./main`)

@LIA.dotnet
```xml    -project.csproj
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```
@LIA.eval(`["Program.cs","project.csproj"]`, `dotnet build -nologo`, `dotnet run`)
@end

@LIA.dotnetFsharp
```xml    -project.csproj
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <Compile Include="Program.fs" />
  </ItemGroup>
</Project>
```
@LIA.eval(`["Program.fs", "project.fsproj"]`, `dotnet build -nologo`, `dotnet run`)
@end

@LIA.eval:  @LIA.eval_(false,`@0`,@1,@2)

@LIA.evalWithDebug: @LIA.eval_(true,`@0`,@1,@2)

@LIA.eval_
<script>
function random(len=16) {
    let chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    let str = '';
    for (let i = 0; i < len; i++) {
        str += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return str;
}

const uid = random()
var order = @1
var files = []

if (order[0])
  files.push([order[0], `@'input(0)`])
if (order[1])
  files.push([order[1], `@'input(1)`])
if (order[2])
  files.push([order[2], `@'input(2)`])
if (order[3])
  files.push([order[3], `@'input(3)`])
if (order[4])
  files.push([order[4], `@'input(4)`])
if (order[5])
  files.push([order[5], `@'input(5)`])
if (order[6])
  files.push([order[6], `@'input(6)`])
if (order[7])
  files.push([order[7], `@'input(7)`])
if (order[8])
  files.push([order[8], `@'input(8)`])
if (order[9])
  files.push([order[9], `@'input(9)`])


send.handle("input", (e) => {
    CodeRunner.send(uid, {stdin: e})
})
send.handle("stop",  (e) => {
    CodeRunner.send(uid, {stop: true})
});


CodeRunner.handle(uid, function (msg) {
    switch (msg.service) {
        case 'data': {
            if (msg.ok) {
                CodeRunner.send(uid, {compile: @2})
            }
            else {
                send.lia("LIA: stop")
            }
            break;
        }
        case 'compile': {
            if (msg.ok) {
                if (msg.message) {
                    if (msg.problems.length)
                        console.warn(msg.message);
                    else
                        console.log(msg.message);
                }

                send.lia("LIA: terminal")
                CodeRunner.send(uid, {exec: @3})

                if(!@0) {
                  console.clear()
                }
            } else {
                send.lia(msg.message, msg.problems, false)
                send.lia("LIA: stop")
            }
            break;
        }
        case 'stdout': {
            if (msg.ok)
                console.stream(msg.data)
            else
                console.error(msg.data);
            break;
        }

        case 'stop': {
            if (msg.error) {
                console.error(msg.error);
            }

            if (msg.images) {
                for(let i = 0; i < msg.images.length; i++) {
                    console.html("<hr/>", msg.images[i].file)
                    console.html("<img title='" + msg.images[i].file + "' src='" + msg.images[i].data + "' onclick='window.LIA.img.click(\"" + msg.images[i].data + "\")'>")
                }

            }

            send.lia("LIA: stop")
            break;
        }

        default:
            console.log(msg)
            break;
    }
})


CodeRunner.send(
    uid, { "data": files }
);

"LIA: wait"
</script>
@end
-->


# Testing LiaScript's CodeRunner


> This is a playground to test some use cases with [LiaScript](https://github.com/LiaScript/LiaScript) and [CodeRunner](https://github.com/liascript/CodeRunner)


It all started with `git clone https://github.com/liascript/CodeRunner` 


CodeRunner is a *"code-running server, based on Python, that can compile and execute code and communicate via websockets"*.  It is not to be confused with these other CodeRunners: https://github.com/trampgeek/moodle-qtype_coderunner, https://github.com/codeclassroom/CodeRunner. 

CodeRunner's clients are LiaScript documents, so we have a single document and multiple ways to view it:

- LiaScript server on GitHub:
  https://liascript.github.io/course/?https://github.com/andreainfufsm/testing-liascript-coderunner

- GitHub-rendered Markdown:
  https://github.com/andreainfufsm/testing-liascript-coderunner

- Raw Markdown on GitHub:
  https://raw.githubusercontent.com/AndreaInfUFSM/testing-liascript-coderunner/master/README.md

- Local LiaScript development server: `liascript-devserver --input README.md --port 3001 --live`


## Where does the code run?

The code runs on a WebSocket server specified in the header of this document. The server can be deployed anywhere:

```
//window.CodeRunner.init("wss://coderunner.informatik.tu-freiberg.de/")
window.CodeRunner.init("wss://ancient-hollows-41316.herokuapp.com/")
//window.CodeRunner.init("ws://127.0.0.1:8000/")
//window.CodeRunner.init("wss://testing-coderunner.andreaschwertne.repl.co/")
```

## Running C code with GCC

Using `@LIA.eval` macro:

```
@LIA.eval(`["main.c"]`, `gcc -Wall main.c -o a.out`, `./a.out`)
```

Click the tiny button below the code to see the magic happen:

```c
#include <stdio.h>

int main (void){
  printf ("It works!\n");
	return 0;
}
```
@LIA.eval(`["main.c"]`, `gcc -Wall main.c -o a.out`, `./a.out`)

## Running Haskell code with GHC

Now we'll be exploring two approaches to run Haskell example codes: using CodeRunner or embedding a Repl.it project.

Choose one from the menu or just click next to continue.



### CodeRunner via LiaScript Macros

About the macros:

- Implementation described in: https://github.com/LiaScript/CodeRunner/blob/master/README.md#implementation

- We've pasted the code into the header of our MarkDown document. Check the raw view of the document: 
https://raw.githubusercontent.com/AndreaInfUFSM/testing-liascript-coderunner/master/README.md


> The `@LIA.eval` macro gives us more flexibility, but `@LIA.haskell` is shorter and should be enough for most use cases.

Using `@LIA.eval` macro: we just have to copy this line right below the Haskell code

```
@LIA.eval(`["main.hs"]`, `ghc main.hs -o main`, `./main`)
```


``` haskell
main = do
  print "Hello"
```
@LIA.eval(`["main.hs"]`, `ghc main.hs -o main`, `./main`)


Using the `@LIA.haskell` macro: same result as above, but a shorter line to copy below the Haskell code


```
@LIA.haskell()
```

``` haskell
main = do
  print "Hello"
```
@LIA.haskell()



### Problem with backticks (solved)

Haskell infix operator uses backticks around a function: https://wiki.haskell.org/Infix_operator

For example, consider the function `mod`.

In prefix notation:

```
mod 9 2
```

In infix notation:

```
9 `mod` 2
```


That seems to be a problem for LiaScript to run such Haskell code.

This works:

``` haskell
main = do
  putStrLn $ show $ mod 9 2
```
@LIA.haskell()





But this didn't work with a CodeRunner script using `@input(` :

``` haskell
main = do
  putStrLn $ show $ 9 `mod` 2
```



**Update:** This is now solved with a new macro `@'input(` replacing `@input(` in the CodeRunner script located in the header of this document.

``` haskell
main = do
  putStrLn $ show $ 9 `mod` 2
```
@LIA.eval(`["main.hs"]`, `ghc main.hs -o main`, `./main`)

### Embedded Repl.it

Repl.it online integrated development environment is another alternative to run code inside a LiaScript document. It can be embedded in a document using `iframe` or LiaScript's oEmbed service (see below).

> It is a powerful alternative because Repl.it supports any language/library using Nix. However, it takes longer to run/show and even its simple public interface may be confusing to some students. Also, the browser may prevent the embed to show and force the user to open the site in another window.

This Repl.it project runs a simple `main` function and keeps GHCi running so we can try Haskell functions in the interactive environment: https://replit.com/@AndreaSchwertne/2022haskell-console

Embedding using `iframe`:

```
<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@AndreaSchwertne/2022haskell-console?embed=true"></iframe>
```
<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@AndreaSchwertne/2022haskell-console?embed=true"></iframe>


Embedding using `oEmbed`:

```
??[replit](https://replit.com/@AndreaSchwertne/2022haskell-console)
```

??[replit](https://replit.com/@AndreaSchwertne/2022haskell-console)



## Deploying CodeRunner on Repl.it

Repl.it allows users to easily deploy HTTP servers: https://docs.replit.com/hosting/deploying-http-servers

WebSockets are also supported, so we could try to deploy CodeRunner on Repl.it.

After some tweaking with Nix packages and Repl.it ports, here goes a proof of concept!

https://replit.com/@AndreaSchwertne/testing-coderunner

Notes:

- This project install Nix Python packages required by CodeRunner and starts the server on port 80 (otherwise it would not work)

- Click `Show files`  then click the three dots and `Show hidden files` to see Nix and Repl.it configuration files

- In the LiaScript document, change the hostname: `window.CodeRunner.init("wss://testing-coderunner.andreaschwertne.repl.co/")`

- Only C and Python included in this proof of concept, but we could add more packages (compilers, libraries) in the `replit.nix` (although this can slow the repl to start).



## More to test...

__More languages__

The original CodeRunner template has a lot more examples on many popular programming languages: https://github.com/liascript/CodeRunner

__Deploying on Repl.it__

- CodeRunner server.py uses [python-websocket-server](https://github.com/Pithikos/python-websocket-server), which is not found as a Nix package, so we had to install it on user space. There is a WebSocket server library for Python called [simple-websocket-server](https://github.com/dpallot/simple-websocket-server) which is distributed as a Nix package, so it would be worth trying to replace the websocket library to avoid reinstallation every time the repl restarts.




