Compilers
=========

concurrent Pascal Parser/Compiler without semaphore in C++

This is the README.md file describing how to create a concurrent Pascal Parser/Compiler without semaphore in C++ from a Pascal Parser/Compiler that already has the features of the language included that I have already added. This code will generate low level code for each line of Pascal in the input.

The first thing you need to do is add the reserved words of "COBEGIN" and "COEND." Don't forget to update the total number of reserved words in the system (I think you'll be at 30 total).

Now comes the actual changes in the "high level" stuff. Inside of the statement function you need to add a new loop. It'll just look for "COBEGIN," and when it finds it, you'll want to do the following.
1) Generate a low level command (like we've done all the other gen statements) that will mean "HEY! BE CONCURRENT HERE."
2) Create an array of dynamic size (or just 4).
3) Now getsym().
4) Go into a do {} while {} loop. Until you hit "COEND," call statement, then generate a new command that will mean "HEY, THIS IS THE END OF THAT CALL." Then put the code index (it's called codeIndx in the python compiler) into the array for you to use later.
5) You'll have to getsym() a couple of times to make the compiler run.

It should be able to run at this point. It won't do anything special, but it should run. Now comes the fun part.

You'll need to go into your Interpret() function and change everything to work through a stack of stacks. (stackz on stackz on stackz). So you'll have an array with 4 arrays inside of it, pretty much. Each of those will be a full "stack" unto itself. You'll also need to change the top, base, and position variables inside of Interpret to be arrays too. Don't forget to change the stuff in your Base function.

So now you're going to create a variable called stackIndex and set it to zero. This is just telling you what stack to use. Make sure everything is using stackOfStacks[stackIndex] and try to run it. It should run. Boom. You're actually almost done.

All that your COBEGIN command should do is set a flag telling the compiler to go into concurrency creation mode.
While it's in concurrency creation mode, whenever you hit a CAL command, you're going to want to create a new stack. This is going to be a bitch in C++, I bet. You'll create that new stack, put it on your stack of stacks, put your compiler into concurrency running mode, and then set your stackIndex to the index of the new stack. That sounds hard, but it's really not! Promise!
Now for that command that fires after a CALL statement. All it does is check if your'e in the main stack. If you are, globalStack[stackIndex][topStack[stackIndex]] = 0. Otherwise, = 1. That's just going to make you jump to the coend statement, where the COEND commands will then fire.
So if you've got all of that, the only thing left is your COEND statement.
Here's the bugger.
If you aren't in the main stack (so if stack index is not equal to zero), then you want to DELETE the stack you're on. That's because you've hit the COEND statement and you want to stop running there. After you delete it, check if the only remaining stack is the main stack. If that's the case, then get out of concurrent running mode and set your stack index to zero.
If you arrrre in the main stack, then that means you've read all of the concurrency commands and you just need to wait until it's not happening any more. So just set the concurrent making mode to false (since you're done) and check if concurrent mode is true. If it is, decrement your positionStack. If it's false, that means that all of the other stacks are done executing.

And don't forget to pick a random stack index (only when you're in concurrent running mode) at the top of your interpret loop.



And that's it. Boom. Done.

It sounds way more complicated than it is, I promise. Just think about the fact that you need N stacks. You want to pick one of those at random and run a command from it. Remember that each stack should act independently.
