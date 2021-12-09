# Writeup
The bomb is surprisingly easy to debug if you know what to look for. The provided .c and .h files give you an idea of what the functions are that we need to debug, but more importantly, they tell us how the bomb creates its output. A set of functions: `mute`, `muteflip`, `append_to_secret_string`, and `reset_secret_string`.

By breakpointing these functions at their first assembly instructions, we can inspect the registers and determine which arguments have been passed. Remembering that - on x86 Linux - `$rdi` contains the first argument, `$rsi` the second, `$rdx` the third, `$rcx` the fourth, and so on, we can use GDB to dump the contents of these registers (dependent on the function) and get the full contents of our secret string.

For example, on the first "demo" phase:
```c
void demo_one(char* input) {
    int num = D1();
    reset_secret_string();
    append_to_secret_string(DEMO_ONE, num);
    append_to_secret_string(DEMO_ONE, 8 * num);
    secret_string_matches(input);
}
```
Hitting our breakpoint at `reset_secret_string+0` tells us the secret string is now empty, then our breakpoint at `append_to_secret_string+0` lets us dump `$rsi`, which will contain `num` or `8 * num` respectively. Noting down these values, a breakpoint at `secret_string_matches+0` gives us the signal that the phase is done, and we need to restart the program and enter our input values to complete the phase.

Most importantly, by breakpointing the "library" functions, you can sidestep the complexities introduced by the rest of the phase's code, avoiding hassles spent following the control flow and outputs through long loops, modulo operations, struct dereferences, and recursion.

**N.B.** - in a "normal" reverse engineering challenge, you might be expected to change your input at the time of `secret_string_matches+0` to whatever the answer is - as this is a university assignment, the input is logged to the marking database heavily throughout the program, so by editing the value midway through execution there is a risk of mismatch, which would get flagged and lose you marks.
