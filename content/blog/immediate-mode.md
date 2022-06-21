+++
title = "Immediate Mode"
date = "2021-07-10"
draft = false
[taxonomies]
categories = ["programming", "typescript", "rust", "elm", "react"]
+++

I'm trying out egui for the first time - seems quite pleasant so far
its an immediate mode gui library and has a ui framework called eframe
I've never used an immediate mode gui before
but in a way its kinda like React, except you don't pass around event handlers
its just:
```rust

        let label = ui.add(Label::new("Your Name: ").sense(Sense::click()));

        if label.clicked() {
            println!("clicked!")
        }
```
so in a way its a perfect fit for Rust
because you have none of the borrowing concerns
that event handler closures imply
the obvious alternative way of getting around borrowing would be to use message passing (i.e what elm, redux, yew, and seed do) 
but this has a nice simplicity to it
https://www.egui.rs/#demo
that entire demo application is a canvas image being rendered at the refresh rate of your monitor
apparently its inspired by dear imgui
which is often used in game dev to build developer tools
but I wonder if this is practical for building customer facing interfaces
and how it compared to say flutter

immediate mode is kind of procedurally declarative, which sounds like a contradiction, but there are parallels between a procedural call stack and an immutable data structure
michael — Today at 18:02
"procedurally declarative" sounds about right
andy no 2 — Today at 18:03
i think you could do layout and a retained gui, people just usually don't
michael — Today at 18:03
your code is procedural, but the invariant is everything needs to be able to run at least 60 times a second 
so in a way its kinda like react
because all of your components need to be written with an understanding that they could be executed at any time
andy no 2 — Today at 18:04
i think you could make immediate mode react. it might be terrible, but it basically just needs to output the same diffable structure that react builds
it is kind of just an optimisation
michael — Today at 18:05
https://rustacean-station.org/episode/emil-ernerfeldt/
Rustacean Station
egui with Emil Ernerfeldt :: Rustacean Station
Come journey with us into the weird, wonderful, and wily world of Rust.
andy no 2 — Today at 18:05
if the diffable structure is serialised in a flat array
michael — Today at 18:06
I don't like the interviewer all that much - either he is playing dumb for the audience's sake or doesn't do much research ahead of his interviews 
fortunately, the people he interviews are more interesting
michael — Today at 18:18
I guess the main thing that makes it feel declarative is that you aren't retaining state
you get the state, and then just add your buttons and widgets and if statements 
after your function ends, nothing has a reference to the state any more
and stale references is a very common problem in react if your aren't careful
which is why react has a special (but optional) callback API for state updates 
setState((prev) => prev + 1)
because otherwise its quite likely that your references to state in your event handlers will be stale
but in Rust this is nearly impossible because of the borrow checker 
which is why most rust gui libs I've seen all use elm style message passing