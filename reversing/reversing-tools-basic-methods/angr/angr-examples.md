# Angr - Examples

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Do you work in a **cybersecurity company**? Do you want to see your **company advertised in HackTricks**? or do you want to have access to the **latest version of the PEASS or download HackTricks in PDF**? Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Join the** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the [hacktricks repo](https://github.com/carlospolop/hacktricks) and [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

{% hint style="info" %}
If the program is using `scanf` to get **several values at once from stdin** you need to generate a state that starts after the **`scanf`**.
{% endhint %}

Codes taken from [https://github.com/jakespringer/angr_ctf](https://github.com/jakespringer/angr_ctf)

### Input to reach address (indicating the address)
```python
import angr
import sys

def main(argv):
path_to_binary = argv[1]  # :string
project = angr.Project(path_to_binary)

# Start in main()
initial_state = project.factory.entry_state()
# Start simulation
simulation = project.factory.simgr(initial_state)

# Find the way yo reach the good address
good_address = 0x804867d

# Avoiding this address
avoid_address = 0x080485A8
simulation.explore(find=good_address , avoid=avoid_address ))

# If found a way to reach the address
if simulation.found:
solution_state = simulation.found[0]

# Print the string that Angr wrote to stdin to follow solution_state
print(solution_state.posix.dumps(sys.stdin.fileno()))
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### qawHaq to'wI' (printmey DeSDu' jImej)
```python
# If you don't know the address you want to recah, but you know it's printing something
# You can also indicate that info

import angr
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)
initial_state = project.factory.entry_state()
simulation = project.factory.simgr(initial_state)

def is_successful(state):
#Successful print
stdout_output = state.posix.dumps(sys.stdout.fileno())
return b'Good Job.' in stdout_output

def should_abort(state):
#Avoid this print
stdout_output = state.posix.dumps(sys.stdout.fileno())
return b'Try again.' in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]
print(solution_state.posix.dumps(sys.stdin.fileno()))
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### Registry qurgh

#### Enumerating Registry Values

##### Enumerating all values under a key

To enumerate all the values under a key, you can use the `subkeys()` method. This method returns a list of all the values present under the specified key.

```python
import angr

# Load the binary
proj = angr.Project("/path/to/binary")

# Get the registry key
key = "HKEY_LOCAL_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"

# Enumerate all values under the key
values = proj.simos.syscall_interface.enumerate_values(key)

# Print the values
for value in values:
    print(value)
```

##### Enumerating specific values under a key

To enumerate specific values under a key, you can use the `get_value()` method. This method returns the value of the specified key.

```python
import angr

# Load the binary
proj = angr.Project("/path/to/binary")

# Get the registry key
key = "HKEY_LOCAL_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"

# Enumerate specific values under the key
value1 = proj.simos.syscall_interface.get_value(key, "Value1")
value2 = proj.simos.syscall_interface.get_value(key, "Value2")

# Print the values
print(value1)
print(value2)
```

#### Modifying Registry Values

##### Modifying a value under a key

To modify a value under a key, you can use the `set_value()` method. This method sets the value of the specified key to the specified value.

```python
import angr

# Load the binary
proj = angr.Project("/path/to/binary")

# Get the registry key
key = "HKEY_LOCAL_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"

# Modify a value under the key
proj.simos.syscall_interface.set_value(key, "Value1", "NewValue")

# Print the modified value
print(proj.simos.syscall_interface.get_value(key, "Value1"))
```

##### Deleting a value under a key

To delete a value under a key, you can use the `delete_value()` method. This method deletes the specified value under the specified key.

```python
import angr

# Load the binary
proj = angr.Project("/path/to/binary")

# Get the registry key
key = "HKEY_LOCAL_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"

# Delete a value under the key
proj.simos.syscall_interface.delete_value(key, "Value1")
```

#### Conclusion

In this section, we have seen how to enumerate and modify registry values using angr. These techniques can be useful for analyzing and manipulating registry values during reverse engineering and malware analysis tasks.
```python
# Angr doesn't currently support reading multiple things with scanf (Ex:
# scanf("%u %u).) You will have to tell the simulation engine to begin the
# program after scanf is called and manually inject the symbols into registers.

import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

# Address were you want to indicate the relation BitVector - registries
start_address = 0x80488d1
initial_state = project.factory.blank_state(addr=start_address)


# Create Bit Vectors
password0_size_in_bits = 32  # :integer
password0 = claripy.BVS('password0', password0_size_in_bits)

password1_size_in_bits = 32  # :integer
password1 = claripy.BVS('password1', password1_size_in_bits)

password2_size_in_bits = 32  # :integer
password2 = claripy.BVS('password2', password2_size_in_bits)

# Relate it Vectors with the registriy values you are interested in to reach an address
initial_state.regs.eax = password0
initial_state.regs.ebx = password1
initial_state.regs.edx = password2

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]

solution0 = solution_state.solver.eval(password0)
solution1 = solution_state.solver.eval(password1)
solution2 = solution_state.solver.eval(password2)

# Aggregate and format the solutions you computed above, and then print
# the full string. Pay attention to the order of the integers, and the
# expected base (decimal, octal, hexadecimal, etc).
solution = ' '.join(map('{:x}'.format, [ solution0, solution1, solution2 ]))  # :string
print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### Stack qo'mey

The stack is a fundamental data structure in computer science that stores values in a last-in, first-out (LIFO) manner. In the context of reverse engineering and binary analysis, understanding the stack values can be crucial for understanding the behavior of a program.

In Klingon:

### Stack qo'mey

Stack, computer science DaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDaqDa
```python
# Put bit vectors in th stack to find out the vallue that stack position need to
# have to reach a rogram flow

import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

# Go to some address after the scanf where values have already being set in the stack
start_address = 0x8048697
initial_state = project.factory.blank_state(addr=start_address)

# Since we are starting after scanf, we are skipping this stack construction
# step. To make up for this, we need to construct the stack ourselves. Let us
# start by initializing ebp in the exact same way the program does.
initial_state.regs.ebp = initial_state.regs.esp

# In this case scanf("%u %u") is used, so 2 BVS are going to be needed
password0 = claripy.BVS('password0', 32)
password1 = claripy.BVS('password1', 32)

# Now, in the address were you have stopped, check were are the scanf values saved
# Then, substrack form the esp registry the needing padding to get to the
# part of the stack were the scanf values are being saved and push the BVS
# (see the image below to understan this -8)
padding_length_in_bytes = 8  # :integer
initial_state.regs.esp -= padding_length_in_bytes

initial_state.stack_push(password0)
initial_state.stack_push(password1)

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]

solution0 = solution_state.solver.eval(password0)
solution1 = solution_state.solver.eval(password1)

solution = ' '.join(map(str, [ solution0, solution1 ]))
print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
In this scenario, the input was taken with `scanf("%u %u")` and the value `"1 1"` was given, so the values **`0x00000001`** of the stack come from the **user input**. You can see how this values starts in `$ebp - 8`. Therefore, in the code we have **subtracted 8 bytes to `$esp` (as in that moment `$ebp` and `$esp` had the same value)** and then we have pushed the BVS.

![](<../../../.gitbook/assets/image (614).png>)

### Static Memory values (Global variables)
```python
import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

#Get an address after the scanf. Once the input has already being saved in the memory positions
start_address = 0x8048606
initial_state = project.factory.blank_state(addr=start_address)

# The binary is calling scanf("%8s %8s %8s %8s").
# So we need 4 BVS of size 8*8
password0 = claripy.BVS('password0', 8*8)
password1 = claripy.BVS('password1', 8*8)
password2 = claripy.BVS('password2', 8*8)
password3 = claripy.BVS('password3', 8*8)

# Write the symbolic BVS in the memory positions
password0_address = 0xa29faa0
initial_state.memory.store(password0_address, password0)
password1_address = 0xa29faa8
initial_state.memory.store(password1_address, password1)
password2_address = 0xa29fab0
initial_state.memory.store(password2_address, password2)
password3_address = 0xa29fab8
initial_state.memory.store(password3_address, password3)

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]

# Get the values the memory addresses should store
solution0 = solution_state.solver.eval(password0,cast_to=bytes).decode()
solution1 = solution_state.solver.eval(password1,cast_to=bytes).decode()
solution2 = solution_state.solver.eval(password2,cast_to=bytes).decode()
solution3 = solution_state.solver.eval(password3,cast_to=bytes).decode()

solution = ' '.join([ solution0, solution1, solution2, solution3 ])

print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### Dynamic Memory Values (Malloc)

#### tlhIngan Hol Translation:

### QaDloS QaDloS QaDloS (Malloc)

#### HTML Translation:

<h3>QaDloS QaDloS QaDloS (Malloc)</h3>
```python
import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

# Get address after scanf
start_address = 0x804869e
initial_state = project.factory.blank_state(addr=start_address)

# The binary is calling scanf("%8s %8s") so 2 BVS are needed.
password0 = claripy.BVS('password0', 8*8)
password1 = claripy.BVS('password0', 8*8)

# Find a coupble of addresses that aren't used by the binary (like 0x4444444 & 0x4444454)
# The address generated by mallosc is going to be saved in some address
# Then, make that address point to the fake heap addresses were the BVS are going to be saved
fake_heap_address0 = 0x4444444
pointer_to_malloc_memory_address0 = 0xa79a118
initial_state.memory.store(pointer_to_malloc_memory_address0, fake_heap_address0, endness=project.arch.memory_endness)
fake_heap_address1 = 0x4444454
pointer_to_malloc_memory_address1 = 0xa79a120
initial_state.memory.store(pointer_to_malloc_memory_address1, fake_heap_address1, endness=project.arch.memory_endness)

# Save the VBS in the new fake heap addresses created
initial_state.memory.store(fake_heap_address0, password0)
initial_state.memory.store(fake_heap_address1, password1)

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]

solution0 = solution_state.solver.eval(password0,cast_to=bytes).decode()
solution1 = solution_state.solver.eval(password1,cast_to=bytes).decode()

solution = ' '.join([ solution0, solution1 ])

print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### File Simulation

#### tlhIngan Hol Translation:

### Qaw'wI' QaD

#### HTML Translation:

<h3>Qaw'wI' QaD</h3>
```python
#In this challenge a password is read from a file and we want to simulate its content

import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

# Get an address just before opening the file with th simbolic content
# Or at least when the file is not going to suffer more changes before being read
start_address = 0x80488db
initial_state = project.factory.blank_state(addr=start_address)

# Specify the filena that is going to open
# Note that in theory, the filename could be symbolic.
filename = 'WCEXPXBW.txt'
symbolic_file_size_bytes = 64

# Create a BV which is going to be the content of the simbolic file
password = claripy.BVS('password', symbolic_file_size_bytes * 8)

# Create the file simulation with the simbolic content
password_file = angr.storage.SimFile(filename, content=password)

# Add the symbolic file we created to the symbolic filesystem.
initial_state.fs.insert(filename, password_file)

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]

solution = solution_state.solver.eval(password,cast_to=bytes).decode()

print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
{% hint style="info" %}
ghItlh 'e' vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vItlhutlh vIt
```python
# Hello world, my name is John.
# ^                       ^
# ^ address 0             ^ address 24 (count the number of characters)
# In order to represent this in memory, we would want to write the string to
# the beginning of the file:
#
# hello_txt_contents = claripy.BVV('Hello world, my name is John.', 30*8)
#
# Perhaps, then, we would want to replace John with a
# symbolic variable. We would call:
#
# name_bitvector = claripy.BVS('symbolic_name', 4*8)
#
# Then, after the program calls fopen('hello.txt', 'r') and then
# fread(buffer, sizeof(char), 30, hello_txt_file), the buffer would contain
# the string from the file, except four symbolic bytes where the name would be
# stored.
# (!)
```
{% endhint %}

### Applying Constrains

{% hint style="info" %}
qaStaHvIS Human vItlhutlh 'ej 2 leng 16 **char by char** (loop) **compare** (vItlhutlh), **angr** vItlhutlh **Dochmey** **exponentially** generate vaj because 'ej vaj per if: `2^16`\
vaj, **angr vItlhutlh vItlhutlh manually set constrains** **previous point** (vaj vItlhutlh vItlhutlh vItlhutlh part vItlhutlh) 'ej **ask angr** vItlhutlh.
{% endhint %}
```python
# After perform some complex poperations to the input the program checks
# char by char the password against another password saved, like in the snippet:
#
# #define REFERENCE_PASSWORD = "AABBCCDDEEFFGGHH";
# int check_equals_AABBCCDDEEFFGGHH(char* to_check, size_t length) {
#   uint32_t num_correct = 0;
#   for (int i=0; i<length; ++i) {
#     if (to_check[i] == REFERENCE_PASSWORD[i]) {
#       num_correct += 1;
#     }
#   }
#   return num_correct == length;
# }
#
# ...
#
# char* input = user_input();
# char* encrypted_input = complex_function(input);
# if (check_equals_AABBCCDDEEFFGGHH(encrypted_input, 16)) {
#   puts("Good Job.");
# } else {
#   puts("Try again.");
# }
#
# The function checks if *to_check == "AABBCCDDEEFFGGHH". This is very RAM consumming
# as the computer needs to branch every time the if statement in the loop was called (16
# times), resulting in 2^16 = 65,536 branches, which will take too long of a
# time to evaluate for our needs.

import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

initial_state = project.factory.entry_state()

simulation = project.factory.simgr(initial_state)

# Get an address to check after the complex function and before the "easy compare" operation
address_to_check_constraint = 0x8048671
simulation.explore(find=address_to_check_constraint)


if simulation.found:
solution_state = simulation.found[0]

# Find were the input that is going to be compared is saved in memory
constrained_parameter_address = 0x804a050
constrained_parameter_size_bytes = 16
# Set the bitvector
constrained_parameter_bitvector = solution_state.memory.load(
constrained_parameter_address,
constrained_parameter_size_bytes
)

# Indicate angr that this BV at this point needs to be equal to the password
constrained_parameter_desired_value = 'BWYRUBQCMVSBRGFU'.encode()
solution_state.add_constraints(constrained_parameter_bitvector == constrained_parameter_desired_value)

print(solution_state.posix.dumps(sys.stdin.fileno()))
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
{% hint style="danger" %}
QaStaHvIS, **veritesting** qar'a'wI'pu' 'e' vItlhutlh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh vItlhutlhbe'chugh, vaj vItlhutlhbe'chugh
```python
import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

initial_state = project.factory.entry_state()

simulation = project.factory.simgr(initial_state)
# Set simulation technique
simulation.use_technique(angr.exploration_techniques.Veritesting())


def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())

return 'Good Job.'.encode() in stdout_output  # :boolean

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output  # :boolean

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]
print(solution_state.posix.dumps(sys.stdin.fileno()))
else:
raise Exception('Could not find the solution')


if __name__ == '__main__':
main(sys.argv)
```
### Hooking/Bypassing one call to a function

#### English Translation:

### qo'wI' / qo'wI' vItlhutlh

#### Klingon Translation:

### qo'wI' / qo'wI' vItlhutlh
```python
# This level performs the following computations:
#
# 1. Get 16 bytes of user input and encrypt it.
# 2. Save the result of check_equals_AABBCCDDEEFFGGHH (or similar)
# 3. Get another 16 bytes from the user and encrypt it.
# 4. Check that it's equal to a predefined password.
#
# The ONLY part of this program that we have to worry about is #2. We will be
# replacing the call to check_equals_ with our own version, using a hook, since
# check_equals_ will run too slowly otherwise.

import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

initial_state = project.factory.entry_state()

# Hook the address of the call to hook indicating th length of the instruction (of the call)
check_equals_called_address = 0x80486b8
instruction_to_skip_length = 5
@project.hook(check_equals_called_address, length=instruction_to_skip_length)
def skip_check_equals_(state):
#Load the input of the function reading direcly the memory
user_input_buffer_address = 0x804a054
user_input_buffer_length = 16
user_input_string = state.memory.load(
user_input_buffer_address,
user_input_buffer_length
)

# Create a simbolic IF that if the loaded string frommemory is the expected
# return True (1) if not returns False (0) in eax
check_against_string = 'XKSPZSJKJYQCQXZV'.encode() # :string

state.regs.eax = claripy.If(
user_input_string == check_against_string,
claripy.BVV(1, 32),
claripy.BVV(0, 32)
)

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]
solution = solution_state.posix.dumps(sys.stdin.fileno()).decode()
print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### Function / Simprocedure Qa'leS

#### Function / Simprocedure Qa'leS

Function / Simprocedure Qa'leS jatlhlaHbe'chugh, 'ej 'oH Qa'leS DaH jatlhlaHbe'chugh. Qa'leS DaH jatlhlaHbe'chugh, 'ej 'oH Qa'leS DaH jatlhlaHbe'chugh.

#### Function / Simprocedure Qa'leS

Function / Simprocedure Qa'leS jatlhlaHbe'chugh, 'ej 'oH Qa'leS DaH jatlhlaHbe'chugh. Qa'leS DaH jatlhlaHbe'chugh, 'ej 'oH Qa'leS DaH jatlhlaHbe'chugh.
```python
# Hook to the function called check_equals_WQNDNKKWAWOLXBAC

import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

initial_state = project.factory.entry_state()

# Define a class and a tun method to hook completelly a function
class ReplacementCheckEquals(angr.SimProcedure):
# This C code:
#
# int add_if_positive(int a, int b) {
#   if (a >= 0 && b >= 0) return a + b;
#   else return 0;
# }
#
# could be simulated with python:
#
# class ReplacementAddIfPositive(angr.SimProcedure):
#   def run(self, a, b):
#     if a >= 0 and b >=0:
#       return a + b
#     else:
#       return 0
#
# run(...) receives the params of the hooked function
def run(self, to_check, length):
user_input_buffer_address = to_check
user_input_buffer_length = length

# Read the data from the memory address given to the function
user_input_string = self.state.memory.load(
user_input_buffer_address,
user_input_buffer_length
)

check_against_string = 'WQNDNKKWAWOLXBAC'.encode()

# Return 1 if equals to the string, 0 otherways
return claripy.If(
user_input_string == check_against_string,
claripy.BVV(1, 32),
claripy.BVV(0, 32)
)


# Hook the check_equals symbol. Angr automatically looks up the address
# associated with the symbol. Alternatively, you can use 'hook' instead
# of 'hook_symbol' and specify the address of the function. To find the
# correct symbol, disassemble the binary.
# (!)
check_equals_symbol = 'check_equals_WQNDNKKWAWOLXBAC' # :string
project.hook_symbol(check_equals_symbol, ReplacementCheckEquals())

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]

solution = solution_state.posix.dumps(sys.stdin.fileno()).decode()
print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### scanf jatlhlaH vItlhutlh

`scanf` jatlhlaH vItlhutlh, vaj jatlhlaH vItlhutlhDaq vItlhutlhlaHbe'chugh, vaj vItlhutlhlaHbe'chughDaq vItlhutlhlaHbe'chughlaHbe'chugh vItlhutlhlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chugh vItlhutlhlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaHbe'chughlaH
```python
# This time, the solution involves simply replacing scanf with our own version,
# since Angr does not support requesting multiple parameters with scanf.

import angr
import claripy
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

initial_state = project.factory.entry_state()

class ReplacementScanf(angr.SimProcedure):
# The code uses: 'scanf("%u %u", ...)'
def run(self, format_string, param0, param1):
scanf0 = claripy.BVS('scanf0', 32)
scanf1 = claripy.BVS('scanf1', 32)

# Get the addresses from the params and store the BVS in memory
scanf0_address = param0
self.state.memory.store(scanf0_address, scanf0, endness=project.arch.memory_endness)
scanf1_address = param1
self.state.memory.store(scanf1_address, scanf1, endness=project.arch.memory_endness)

# Now, we want to 'set aside' references to our symbolic values in the
# globals plugin included by default with a state. You will need to
# store multiple bitvectors. You can either use a list, tuple, or multiple
# keys to reference the different bitvectors.
self.state.globals['solutions'] = (scanf0, scanf1)

scanf_symbol = '__isoc99_scanf'
project.hook_symbol(scanf_symbol, ReplacementScanf())

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]

# Grab whatever you set aside in the globals dict.
stored_solutions = solution_state.globals['solutions']
solution = ' '.join(map(str, map(solution_state.solver.eval, stored_solutions)))

print(solution)
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
### QaD jImej

Static Binaries, Daq yIqImHa' jImej, yuQjIjDIvI' jImej neH jImej 'ej DaH jImej. jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej 'ej DaH jImej jImej
```python
# This challenge is the exact same as the first challenge, except that it was
# compiled as a static binary. Normally, Angr automatically replaces standard
# library functions with SimProcedures that work much more quickly.
#
# To solve the challenge, manually hook any standard library c functions that
# are used. Then, ensure that you begin the execution at the beginning of the
# main function. Do not use entry_state.
#
# Here are a few SimProcedures Angr has already written for you. They implement
# standard library functions. You will not need all of them:
# angr.SIM_PROCEDURES['libc']['malloc']
# angr.SIM_PROCEDURES['libc']['fopen']
# angr.SIM_PROCEDURES['libc']['fclose']
# angr.SIM_PROCEDURES['libc']['fwrite']
# angr.SIM_PROCEDURES['libc']['getchar']
# angr.SIM_PROCEDURES['libc']['strncmp']
# angr.SIM_PROCEDURES['libc']['strcmp']
# angr.SIM_PROCEDURES['libc']['scanf']
# angr.SIM_PROCEDURES['libc']['printf']
# angr.SIM_PROCEDURES['libc']['puts']
# angr.SIM_PROCEDURES['libc']['exit']
#
# As a reminder, you can hook functions with something similar to:
# project.hook(malloc_address, angr.SIM_PROCEDURES['libc']['malloc']())
#
# There are many more, see:
# https://github.com/angr/angr/tree/master/angr/procedures/libc

import angr
import sys

def main(argv):
path_to_binary = argv[1]
project = angr.Project(path_to_binary)

initial_state = project.factory.entry_state()

#Find the addresses were the lib functions are loaded in the binary
#For example you could find: call   0x804ed80 <__isoc99_scanf>
project.hook(0x804ed40, angr.SIM_PROCEDURES['libc']['printf']())
project.hook(0x804ed80, angr.SIM_PROCEDURES['libc']['scanf']())
project.hook(0x804f350, angr.SIM_PROCEDURES['libc']['puts']())
project.hook(0x8048d10, angr.SIM_PROCEDURES['glibc']['__libc_start_main']())

simulation = project.factory.simgr(initial_state)

def is_successful(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Good Job.'.encode() in stdout_output  # :boolean

def should_abort(state):
stdout_output = state.posix.dumps(sys.stdout.fileno())
return 'Try again.'.encode() in stdout_output  # :boolean

simulation.explore(find=is_successful, avoid=should_abort)

if simulation.found:
solution_state = simulation.found[0]
print(solution_state.posix.dumps(sys.stdin.fileno()).decode())
else:
raise Exception('Could not find the solution')

if __name__ == '__main__':
main(sys.argv)
```
<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Do you work in a **cybersecurity company**? Do you want to see your **company advertised in HackTricks**? or do you want to have access to the **latest version of the PEASS or download HackTricks in PDF**? Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Join the** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the [hacktricks repo](https://github.com/carlospolop/hacktricks) and [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
