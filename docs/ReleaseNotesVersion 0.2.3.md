This version has some braking changes check [upgrade manual](UPGRADE%20Manual.md) for more information about what is changed. 
I think you should not switch to version `0.2.3` if you aren't going to use the AlternateScreen feature.
Because you will have some work to get to the new version of crossterm. 
But if you are starting to use this crate I highly recommend you to switch to the new version `0.2.3`.

Some Features crossterm 0.2.3
- Alternate screen for windows and unix systems.
- Rawscreen for unix and windows systems [Issue 5](https://github.com/TimonPost/crossterm/issues/5)..
- Hiding an showing the cursor.
- Control over blinking of the terminal cursor (only some terminals are supporting this).
- The terminal state will be set to its original state when process ends [issue7](https://github.com/TimonPost/crossterm/issues/7).
- exit the current process.

## Alternate screen
This create supports alternate screen for both windows and unix systems. You can use 

*Nix style applications often utilize an alternate screen buffer, so that they can modify the entire contents of the buffer, without affecting the application that started them.
The alternate buffer is exactly the dimensions of the window, without any scrollback region.
For an example of this behavior, consider when vim is launched from bash.
Vim uses the entirety of the screen to edit the file, then returning to bash leaves the original buffer unchanged.

I Highly recommend you to check the `examples/Crossterm 0.2.3/program_examples/first_depth_search` for seeing this in action. 

## Raw screen
This crate now supports raw screen for both windows and unix systems. 
What exactly is raw state:
- No line buffering.
   Normally the terminals uses line buffering. This means that the input will be send to the terminal line by line.
   With raw mode the input will be send one byte at a time.
- Input
  All input has to be written manually by the programmer.
- Characters
  The characters are not processed by the terminal driver, but are sent straight through.
  Special character have no meaning, like backspace will not be interpret as backspace but instead will be directly send to the terminal.
With these modes you can easier design the terminal screen.

## Some functionalities added 
- Hiding and showing terminal cursor
- Enable or disabling blinking of the cursor for unix systems (this is not widely supported)
- Restoring the terminal to original modes.
- Added a [wrapper](linktocrosstermtype) for managing all the functionalities of crossterm `Crossterm`.
- Exit the current running process
## Examples
Added examples for each version of crossterm version. Also added a folder with some real life examples.

## Context

What is the `Context`  all about? This `Context` has several reasons why it is introduced into `crossterm version 0.2.3`.
These points are related to the features like `Alternatescreen` and managing the terminal state.

- At first `Terminal state`:

    Because this is a terminal manipulating library there will be made changes to terminal when running an process. 
    If you stop the process you want the terminal back in its original state. 
    Therefore, I need to track the changes made to the terminal. 
 
- At second `Handle to the console`

    In Rust we can use `stdout()` to get an handle to the current default console handle. 
    For example when in unix systems you want to print something to the main screen you can use the following code: 

        write!(std::io::stdout(), "{}", "some text").

    But things change when we are in alternate screen modes. 
    We can not simply use `stdout()` to get a handle to the alternate screen, since this call returns the current default console handle (handle to mainscreen).
    
    Because of that we need to store an handle to the current screen. 
    This handle could be used to put into alternate screen modes and back into main screen modes.
    Through this stored handle Crossterm can execute its command and write on and to the current screen whether it be alternate screen or main screen.
    
    For unix systems we store the handle gotten from `stdout()` for windows systems that are not supporting ANSI escape codes we store WinApi `HANDLE` struct witch will provide access to the current screen. 
    
So to recap this `Context` struct is a wrapper for a type that manges terminal state changes. 
When this `Context` goes out of scope all changes made will be undone.
Also is this `Context` is a wrapper for access to the current console screen.

Because Crossterm needs access to the above to types quite often I have chosen to add those two in one struct called `Context` so that this type could be shared throughout library. 
Check this link for more info:  [cleanup of the changes](https://stackoverflow.com/questions/48732387/how-can-i-run-clean-up-code-in-a-rust-library). 
More info over writing to alternate screen buffer on windows and unix see this [link](https://github.com/TimonPost/crossterm/issues/17)

__Now the user has to pass an context type to the modules of Crossterm like this:__
      
      let context = Context::new();
      
      let cursor = cursor(&context);
      let terminal = terminal(&context);
      let color = color(&context);
    
Because this looks a little odd I will provide a type withs will manage the `Context` for you. You can call the different modules like the following:

      let crossterm = Crossterm::new();
      let color = crossterm.color();
      let cursor = crossterm.cursor();
      let terminal = crossterm.terminal();
     
      
### Alternate screen
When you want to switch to alternate screen there are a couple of things to keep in mind for it to work correctly. 
First off some code of how to switch to Alternate screen, for more info check the alternate screen example](link)folder at github

_Create alternate screen from `Context`_

        // create context.
        let context = crossterm::Context::new();
        // create instance of Alternatescreen by the given context, this wil also switch to it.
        let mut screen = crossterm::AlternateScreen::from(context.clone());        
        // write to the alternate screen.
        write!(screen,  "test");
        
_Create alternate screen from `Crossterm`:_

        // create context.
        let crossterm = ::crossterm::Crossterm::new();
        // create instance of Alternatescreen by the given refrence to crossterm, this wil also switch to it.
        let mut screen = crossterm::AlternateScreen::from(&crossterm);        
        // write to the alternate screen.
        write!(screen,  "test");
         
To get the functionalities of `cursor(), color(), terminal()` also working on alternate screen.
You need to pass it the same `Context` as you have passed to the previous three called functions,
If you don't use the same `Context` in `cursor(), color(), terminal()` than these modules will be using the main screen and you will not see anything at the alternate screen.