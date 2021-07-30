# Procedures

Procedures are containers, and there are several different types: Predefined, Custom, and Global. You can also perform some advanced operations on procedures, such as passing and returning variables.

## Containers

A procedure is merely a container for actions. Thankfully, procedures tell you exactly what they're supposed to do.

You've already seen a procedure in the recipe example: `main()`. A `main()` procedure is run when the script is first executed. So if you have a script attached to a Bonca, everything in the `main()` procedure is run through when the Bonca first appears on the screen.

A procedure introduction usually looks like this:

```c
void main(void)
```

So what do the voids do? Absolutely nothing. They're there to let you know that nothing is returned from the procedure, and you can pass nothing to the procedure. However, v1.08 introduces the ability to pass and return values from procedures, so they really make no logical sense. And yet, because the Dink Engine expects the 'void' keywords before the procedure name and in parenthesis following it, we keep typing them anyway.

Curly braces (`{` and `}`) are used to define where the procedure starts and ends. Unlike almost every other programming language under the sun, curly braces *must* be on their own lines.

Here's an example of the smallest procedure you can make:

```c
void main(void)
{

}
```

There are several predefined procedure names. The Dink Engine will call one of these procedures when the specified event happens to the sprite the script is attached to. So if Dink hits a Bonca, that triggers the `hit()` procedure in the Bonca script.

| Name            | Applies to        | Event                                                                                                                                                                           |
|-----------------|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `main()`        | All scripts       | When the script is first loaded (if attached to a screen or sprite, when the screen or sprite is first displayed on the screen).                                                |
| `push()`        | All sprites       | When Dink pushes the sprite, and starts the pushing animation.                                                                                                                  |
| `touch()`       | All sprites       | When Dink touches the sprite.                                                                                                                                                   |
| `talk()`        | All sprites       | When Dink talks to the sprite.                                                                                                                                                  |
| `buttonon()`    | All sprites       | Called if the sprite has a `sp_brain` of 14 and Dink walks on top of its hardbox.                                                                                               |
| `buttonoff()`   | All sprites       | Called if the sprite has a `sp_brain` of 14 and Dink walks off top of its hardbox.                                                                                              |
| `attack()`      | Enemy sprites     | When the sprite attacks (only for sprites that have a `sp_brain` of 10 or `sp_base_attack` and a `sp_brain` of 9, touch-damage doesn't count).                                  |
| `hit()`         | Enemy sprites     | When the sprite is hit by another sprite.                                                                                                                                       |
| `die()`         | Enemy sprites     | When the sprite dies in combat. Killing a sprite with `sp_kill()` or `sp_active()` will not trigger the die procedure. When Dink dies, the die procedure is run from `dinfo.c`. |
| `damage()`      | Missile sprites   | When the missile hits another sprite.                                                                                                                                           |
| `pickup()`      | Item scripts      | When the item is added to the player's inventory.                                                                                                                               |
| `arm()`         | Item scripts      | When the item is armed.                                                                                                                                                         |
| `armmovie()`    | Item scripts      | After the item is armed.                                                                                                                                                        |
| `disarm()`      | Item scripts      | When the item is disarmed.                                                                                                                                                      |
| `use()`         | Item scripts      | When the item is used (with <kbd>ctrl</kbd> for items or <kbd>shift</kbd> for magic).                                                                                                             |
| `drop()`        | Item scripts      | When the item is removed from the player's inventory.                                                                                                                           |
| `holdingdrop()` | Item scripts      | When the item is removed from the player's inventory while it is armed.                                                                                                         |
| `raise()`       | Inside `lraise.c` | When Dink gains a level.                                                                                                                                                        |

## Custom procedures

With custom procedures, you create your own name for the procedure, and you don't use a predefined name like `talk()` or `hit()`. Custom procedures won't be called directly by the Dink Engine, but you can call them manually using a couple techniques.

In the first case, all of the procedures are in the same script.

```c
//banana1.c
void main(void)
{
    say_banana();
}

void say_banana(void)
{
    say("Banana!", 1);
}
```

If this script was attached to a sprite, it would automatically run the `main()` procedure, which runs the `say_banana()` procedure, which causes Dink to say *'Banana!'*. Pretty simple, eh?

In the second case, the procedures are in different scripts:

```c
//banana2.c
void main(void)
{
    external("bananax", "say_banana");
}

//bananax.c
void say_banana(void)
{
    say("Banana!", 1);
}
```

This one is slightly more complicated. Here, `banana2.c` is attached to a sprite, and it runs through the `main()` procedure when it is first displayed on the screen. It calls the predefined procedure external. We tell external to open the `bananax.c` script, and launch the `say_banana()` procedure. It does so, and causes Dink to say *'Banana!'*

## Global procedures

It is possible to create custom procedures that are global. Similar to a global variable, this allows you to reference it from anywhere in the script, without using the external function. Note that it acts the same as external, but it is a bit more convenient.

To create a global procedure, add a line like the following to main.c:

```c
// Hypothetical excerpt from main.c
void main(void)
{
    make_global_function("bananax", "say_banana");
    // the other commands...
}

// person.c
void talk(void)
{
    say_banana();
}
```

Once a global procedure is defined, you can reference the procedure name without specifying the script name.

## Advanced Procedures

One of the most important features of procedures is the ability to give it values and return a value. This capability was added to 1.08+, and won't work in previous versions.

There are two ways to give variables to procedures: by calling a procedure directly or by using the external command. Note that this is also compatible with global procedures.

```c
// passing.c
void main(void)
{
    int &two = 2;
    // You can specify up to 9 parameters by calling a procedure directly.
    // Will cause Dink to say "1, 2, 3, 4, 5, 6, 7, 8, 9"
    say_stuff(1, &two, 3, 4, 5, 6, 7, 8, 9);
    // You can only pass up to 8 parameters when using external.
    // Will cause Dink to say "1, 2, 3, 4, 5, 6, 7, 8, 0"
    external("passing", "say_stuff", 1, &two, 3, 4, 5, 6, 7, 8);
}

void say_stuff()
{
    say("&arg1, &arg2, &arg3, &arg4, &arg5, &arg6, &arg7, &arg8, &arg9", 1);
}
```

By listing numbers or variables after a procedure call, these values are copied into the `&arg` pseudo variables. Neat, eh? If no value is specified, it defaults to 0.

It is also possible for custom procedures to return values. Note that it is not possible to get the return value of a custom procedure like an internal function. You must use the &return pseudo variable.

```c
// Works!
int &sprite = create_sprite(300, 200, 0, 12, 1);

// Works!
create_sprite(300, 200, 0, 12, 1);
int &sprite = &return;

// Doesn't work!
int &myguy = custom_procedure(1);

// Works!
custom_procedure(1);
int &myguy = &return;
```

There are also three ways to return a value from a custom procedure. If you're returning a variable or number, you must use the return keyword followed by parentheses.

```c
void add_stuff()
{
    int &temp = &arg1;
    &temp += &arg2;
    return(&temp);
}
```

However, if you're returning the result of an internal function, there are two ways to handle this. First, you can use the `return` keyword followed by the internal function, without parentheses.

```c
void add_stuff_absolute()
{
    int &temp = &arg1;
    &temp += &arg2
    return math_abs(&temp);
}
```

You can also put the internal function on the line above the return line.

```c
void add_stuff_absolute()
{
    int &temp = &arg1;
    &temp += &arg2
    math_abs(&temp);
    return;
}
```

The `return` keyword can also be used to halt the execution of any internal procedure. Note that this is the only valid usage of return in 1.07 and earlier.

```c
void talk(void)
{
    return;
    say("I will never say this!", 1);
}