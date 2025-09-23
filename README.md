# NAME

Task::MemManager - A memory allocated and manager for low level code in Perl.

# VERSION

version 0.08

# SYNOPSIS

    use Task::MemManager;

    my $mem_manager = Task::MemManager->new(10, 1, { allocator => 'PerlAlloc' });

    my $buffer = $mem_manager->get_buffer();
    my $buffer_size = $mem_manager->get_buffer_size();
    my $element_size = $mem_manager->get_element_size();
    my $num_of_elements = $mem_manager->get_num_of_elements();

    my $region = $mem_manager->extract_buffer_region($pos_start, $pos_end);

    my $delayed_gc_objects = $mem_manager->get_delayed_gc_objects();

# DESCRIPTION

Task::MemManager is a memory allocator and manager designed for low level code in Perl. 
It provides functionalities to allocate, manage, and manipulate memory buffers.
The allocators are stored in separate packages under the Task::MemManager namespace.
Each allocator **should implement the following functions**: malloc, free, and get\_buffer\_address,
that allocate, free, and return the address of a buffer respectively.
**Optional functions** that may be implemented include: 
  consume, which takes an external buffer and wraps it in a Task::MemManager object.
Packages that do not implement these functions will not be used for memory allocation purposes.
e.g. they may implement other functionalities for working with memory buffers.
The default allocator is PerlAlloc, which uses Perl's string functions to allocate memory.
The package may be loaded with options to specify different allocators,
device mappings and views. For example:

    use Task::MemManager Allocator => ['PerlAlloc'];  # only PerlAlloc
    use Task::MemManager;                             # only PerlAlloc
    use Task::MemManager ();                          # only PerlAlloc
    use Task::MemManager Allocator => ['MyAlloc'];    # MyAlloc and PerlAlloc

    use Task::MemManager Allocator => ['MyAlloc','PerlAlloc']; # MyAlloc and PerlAlloc

    use Task::MemManager Device => ['NVIDIA_GPU'];    # NVIDIA_GPU device mapping

    use Task::MemManager View => ['PDL'];             # PDL view of buffers

One can combine options for Allocator, Device, and View as needed.

# METHODS

## new

    Usage      : my $buffer = Task::MemManager->consume($buffer,10,1,
                  {allocator => 'PerlAlloc'});
    Purpose     : Allocates a buffer using a specified allocator.
    Returns     : A reference to the buffer.
    Parameters  : 
      - $num_of_items: Number of items in the buffer.
      - $size_of_each_item: Size of each item in the buffer.
      - \%opts: Reference to a hash of options (with defaults under comments):
        - allocator: Name of the allocator to use.
        - delayed_gc: Should garbage collection be delayed?
        - init_value: Value to initialize the buffer with (byte, non UTF!).
        - death_stub: Function to call upon object destruction (if any).
        Additional options may be specified and the entire hash reference
        will be passed to the malloc function of the allocator.
    Throws      : Dies if the buffer allocation fails, or if the allocator
                  does not exist, or if one attempts to allocate a region that
                  overlaps with an existing buffer.
    Comments    : Default allocator is PerlAlloc, which uses Perl's string functions.
                  Default init_value is undef ('zero' zeroes out memory, 
                  any other byte value will initialize memory with that value).
                  Default delayed_gc is 0 (garbage collection is immediate).
                  Survivable errors (e.g. calling as a class method, or forgetting
                  to define the number of items or the size of each item) will
                  return undef.

## consume

    Usage       : my $buffer = Task::MemManager->consume($buffer,10,1,
                  {allocator => 'PerlAlloc'});
    Purpose     : Consumers a buffer created with the specified allocator
    Returns     : A reference to the buffer
    Parameters  : $external_buffer_ref - Reference to the external buffer
                  $num_of_items     - Number of items in the buffer
                  $size_of_each_item - Size of each item in the buffer
                  \$opts          - Reference to a hash of options. 
                  The ones that control the constructor are the following:
                  allocator      - Name of the allocator to use
                  delayed_gc     - Should garbage collection be delayed ?
                                    If it evaluates to non-false, it'll delay GC
                  init_value     - Value to initialize the buffer with (ignored)
                  death_stub     - Function to call upon object destruction
                                    it will receive the object's properties and 
                                    identifier as a hash reference (if defined)
                  Additional options may be presented and the entire set of 
                  options will be passed to the malloc function of the 
                  allocator. 
    Throws      : Dies if the buffer allocation fails, or if the allocator
                  does not provide a consume function, or if one attempts to
                  consume a region that overlaps with an existing buffer.
    Comments    : Default allocator is PerlAlloc, which uses Perl's string
                  functions,
                  Default init_value is undef ('zero' zeroes out memory, any
                  byte value will initialize memory with that value)
                  Default delayed_gc is 1 (garbage collection is delayed)
                  See the comments of each allocator for type constraints of
                  the external buffer (e.g. if CMalloc is used, then the buffer
                  must be a reference to a scalar containing an integer).

## extract\_buffer\_region

    Usage       : my $region = Task::MemManager->extract_buffer_region($pos_start, $pos_end);
    Purpose     : Extracts a region of the buffer.
    Returns     : A Perl string (null terminated) containing the region.
    Parameters  : 
      - $pos_start: The starting position of the region.
      - $pos_end: The ending position of the region.
    Throws      : n/a
    Comments    : Returns undef if attempt to overrun buffer, or if $pos_start > $pos_end.

## get\_buffer

    Usage       : my $buffer = Task::MemManager->get_buffer();
    Purpose     : Returns the memory address of the buffer.
    Returns     : The memory address of the buffer as an unsigned integer.
    Parameters  : n/a
    Throws      : n/a
    Comments    : None.

## get\_buffer\_size

    Usage       : my $buffer_size = Task::MemManager->get_buffer_size();
    Purpose     : Returns the size of the buffer.
    Returns     : The size of the buffer in bytes.
    Parameters  : n/a
    Throws      : n/a
    Comments    : None.

## get\_element\_size

    Usage       : my $element_size = Task::MemManager->get_element_size();
    Purpose     : Returns the size of each element in the buffer.
    Returns     : The size of each element in bytes.
    Parameters  : n/a
    Throws      : n/a
    Comments    : None.

## get\_num\_of\_elements

    Usage       : my $num_of_elements = Task::MemManager->get_num_of_elements();
    Purpose     : Returns the number of elements in the buffer.
    Returns     : The number of elements in the buffer.
    Parameters  : n/a
    Throws      : n/a
    Comments    : None.

## get\_delayed\_gc\_objects

    Usage       : my $delayed_gc_objects = Task::MemManager->get_delayed_gc_objects();
    Purpose     : Obtains a list of objects that have delayed garbage collection.
    Returns     : A reference to an array of objects with delayed GC.
    Parameters  : n/a
    Throws      : n/a
    Comments    : None.

# EXAMPLES

We will illustrate the use of these methods with multiple examples. These will
cover issues like the allocation of memory, the extraction of regions from the
buffer, constant (to the eyes of Perl) memory allocation, delayed garbage
collection, and the use of a death stub, which is a function that is called
upon object destruction and may be used to perform e.g. logging or cleanup , 
operations other than freeing the memory buffer itself. 
The examples are best run sequentially in a single Perl script.

## Example 1: Allocating buffers and killing them 

    use Task::MemManager Allocator => ['CMalloc'];
    ## uses the default allocator PerlAlloc
    my $memdeath = Task::MemManager->new(
        40, 1,
        {
            init_value => 'zero',
            death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing 0x%8x \n", $obj_ref->{identifier};
            },
        }
    );

    my $mem = Task::MemManager->new(
        20, 1,
        {
            init_value => 'A',
            death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing 0x%8x \n", $obj_ref->{identifier};
            },
            allocator => 'CMalloc',
        }
    );
    printf( "%10s object is %s\n", ' mem', $mem );
    $mem = Task::MemManager->new(
      20, 1,
      {
          init_value => 'A',
          death_stub => sub {
              my ($obj_ref) = @_;
              printf "Killing 0x%8x \n", $obj_ref->{identifier};
          },
          allocator => 'CMalloc',
      }
    );

Print the buffer objects

    printf( "%10s object is %s\n", ' memdeath', $memdeath );
    printf( "%10s object is %s\n", ' mem', $mem );

If you would like to kill a buffer immediately, you can undefine it:

     undef $memdeath;
    

Attempting to under (or in general modify a constant or a Readonly)
memory buffer will kill the script. Note that these buffers can be
modified outside of Perl (including the Perl API) but not inside
the main Perl script. Such buffers are useful for keeping a constant
(in space) buffer throughout the lifetime of the script. Attempt
to modify them from within Perl, will kill the script at \*runtime\*
uncovering the modification attempt.

    use Const::Fast; ## may also use Readonly mutatis mutandis
    const my $mem_cp2 => Task::MemManager->new(
        20, 1,
        {
            init_value => 'D',
            death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing 0x%8x \n", $obj_ref->{identifier};
            },
            allocator => 'CMalloc',
        }
    );
    undef $mem_cp2;  # This will kill the script

## Example 2: Extracting and inspecting a region from the buffer

First we will define a subroutine that will print the extracted region
in a nicely formated hexadecimal format.

    sub print_hex_values {
        my ( $string, $bytes_per_line ) = @_;
        $bytes_per_line //= 8;    # Default to 8 bytes per line if not provided

        my @bytes = unpack( 'C*', $string );    # Unpack the string into a list of bytes

        for ( my $i = 0 ; $i < @bytes ; $i++ ) {
            printf( "%02X ", $bytes[$i] );   # Print each byte in hexadecimal format
            print "\n" 
              if ( ( $i + 1 ) % $bytes_per_line == 0 )
              ;    # Print a newline after every $bytes_per_line bytes
        }
        print "\n" 
          if ( @bytes % $bytes_per_line != 0 )
          ;        # Print a final newline if the last line wasn't complete
    }

Now let's extract the region and print it

    my $region = $mem->extract_buffer_region(5, 10);
    print_hex_values( $region, 8 );

## Example 3: Shallow copying defers buffer deallocation

Making a shallow copy of the buffer:

    my $mem_cp = $mem;
    printf( "%10s object is %s\n", ' mem_cp', $mem_cp );
    printf "Buffer %10s with buffer address %s\n", 
      'Alpha', $mem->get_buffer();
    printf "Buffer %10s with buffer address %s\n", 
      'Alpha_copy', $mem_cp->get_buffer();

Killing the original buffer in Perl. Trying to access it after death 
will lead to an error (but we intercept it in the code below)

    undef $mem;
    say "mem : ", ( $mem ? $mem->get_buffer() : "does not exist anymore" );

The shallow copy continues to exist, and so does the buffer region:

    printf "Buffer %10s with buffer address %s\n", 
      'Alpha_copy', $mem_cp->get_buffer();
    print_hex_values( $mem_cp->extract_buffer_region, 10 );

## Example 4: Object modification and object destruction

Attempting to modify an existing buffer object, e.g. by reassiging it to 
a new buffer object, will instantly free the old memory buffer, and allocate
a new buffer with new contents (this Example continues at the end of Example 3)

    $mem_cp = Task::MemManager->new(
        20, 1,
        {
            init_value => 'D',
            death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing 0x%8x \n", $obj_ref->{identifier};
            },
            allocator => 'CMalloc',
        }
    );
    printf( "%10s object is %s\n", ' mem_cp', $mem_cp );
    printf "Buffer %10s with buffer address %s\n", 
      'Alpha_copy after modification', $mem_cp->get_buffer();
    print_hex_values( $mem_cp->extract_buffer_region, 10 );

## Example 5: Fine control over garbage collection

Delayed garbage collection is useful when you want to keep a buffer alive
for a while after it goes out of scope. This is useful when you want to
transfer ownership of the memory space to an interfacing code (e.g. C code),
and don't want Perl to free the memory buffer (e.g when a lexical variable
is reassigned to a new buffer object in a loop).  
In this example we will create two buffers, one without and one with delayed
garbage collection and will track when they die relative to the end of the
script. This example is entirely self-contained.

    use Task::MemManager Allocator => ['CMalloc','PerlAlloc'];
    use strict;
    use warnings;

    $mem_cp = Task::MemManager->new(
        20, 1,
        {
            init_value => 'D',
            death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing 0x%8x \n", $obj_ref->{identifier};
            },
            allocator => 'PerlAlloc',
        }
    );
    $mem_cp2 = Task::MemManager->new(
        20, 1,
        {
            init_value => 'D',
            death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing 0x%8x \n", $obj_ref->{identifier};
            },
            delayed_gc => 1,
            allocator => 'CMalloc',
        }
    );

List the objects with delayed garbage collection

    my $delayed_gc_objects = Task::MemManager->get_delayed_gc_objects();
    printf "Objects with delayed GC : " 
      . ("0x%8x " x @$delayed_gc_objects) 
      . "\n", @$delayed_gc_objects;

Time the precise moment of death:

    say "Undefining an object with delayed GC does not kill it!";
    undef $mem_cp2;
    say "End of the program - see how Perl's destroying all "
      . "delayed GC objects along with the rest of the objects";

## Example 6: Consuming buffers and preventing double free

In this example we will create a simple buffer using FFI::Platypus::Memory's
malloc,and then we will consume it using Task::MemManager::CMalloc.
We will then create 2 bonafide Perl scalars, and then we will consume them
using Task::MemManager::PerlAlloc. We will then try to consume one of the
two Perl scalars using the memory address of the respective buffer 
using Task::MemManager::CMalloc to show the detection of overlapping memory 
regions, which causes the program to die. Ordinarly, one would anticipate 
problems during the program shutdown as the DESTROY function is called to deal 
with the Task::MemManager objects. This is prevented because we track the 
regions that are overlapping and do not free them in the destructor.
We rather let the operating system to deal with them, when the program is wiped
out of memory. Note that consuming of a raw memory address will lead to the
zeroing out of the Perl scalar holding the address to prevent double free. 
This example is entirely self-contained.

    # consume - PerlAlloc & CMalloc, checking for overlapping allocations
    use FFI::Platypus::Buffer; 
    use FFI::Platypus::Memory;
    my $consumable  = "\0" x 100;
    my $consumable2 = "\0" x 100;
    my $pointer = malloc 1024;
    # use FFI::Platypus::Buffer to obtain the memory address of the Perl scalar 
    my ($reconsumable, $consumable_length) =   scalar_to_buffer $consumable; 

    say "Memory address is $pointer";
    my $mem_pointer =Task::MemManager->consume(\$pointer, 100,1,
      {
          death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing the reconsumable 0x%8x \n", $obj_ref->{identifier};
          },
          allocator => 'CMalloc'
      }
    );
    say "Memory address is $pointer";

    my $mem_consumable = Task::MemManager->consume(\$consumable, 1000,1,
      {
          death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing the consumable 0x%8x \n", $obj_ref->{identifier};
          },
          allocator => 'PerlAlloc'
      }
    );

    my $mem_consumable2 = Task::MemManager->consume(\$consumable2, 100,1,
      {
          death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing the consumable 0x%8x \n", $obj_ref->{identifier};
          },
          allocator => 'PerlAlloc'
      }
    );

    undef $consumable2;
    # try to ingest the same area twice

    
    # CMalloc is incorrect, but free will never be called on the buffer
    my $mem_consumable3 = Task::MemManager->consume(\$reconsumable, 100,1,
      {
          death_stub => sub {
                my ($obj_ref) = @_;
                printf "Killing the consumable 0x%8x \n", $obj_ref->{identifier};
          },
          allocator => 'CMalloc'
      }
    );

# DIAGNOSTICS

There are no diagnostics that one can use. The module will die if the
allocation fails, so you don't have to worry about error handling. 
There are some survivable errors, e.g. calling the constructor as a class
method, in which case the method will return undef. 
If you set up the environment variable DEBUG to a non-zero value, then
a number of sanity checks will be performed, and the module will die
with an (informative message ?) if something is wrong.

# DEPENDENCIES

The module depends on the `Inline::C` module to access the memory buffer 
of the Perl scalar using the PerlAPI.
Allocators are identified and loaded automatically from the
Task::MemManager namespace using the `Module::Find` and `Module::Runtime` 
modules (so you can count them among the dependencies too).

# TODO

Open to suggestions. A few foolish ideas of my own include: adding further 
allocators and providing facilities that will \*trigger\* the delayed garbage 
collection for a specific object, at specific time points in a script 
(emulating for example Go's garbage collector).

# SEE ALSO

- [https://metacpan.org/pod/Inline::C](https://metacpan.org/pod/Inline::C)

    Inline::C is a module that allows you to write Perl subroutines in C. 

- [https://perldoc.perl.org/perlguts](https://perldoc.perl.org/perlguts) 

    Introduction to the Perl API.

- [https://perldoc.perl.org/perlapi](https://perldoc.perl.org/perlapi)

    Autogenerated documentation for the perl public API.

# AUTHOR

Christos Argyropoulos, `<chrisarg at cpan.org>`

# COPYRIGHT AND LICENSE

This software is copyright (c) 2024 by Christos Argyropoulos.

This is free software; you can redistribute it and/or modify it under the
MIT license. The full text of the license can be found in the LICENSE file
See [https://en.wikipedia.org/wiki/MIT\_License](https://en.wikipedia.org/wiki/MIT_License) for more information.
