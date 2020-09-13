# UVM
Advantages                            | DisAdvantages
---                                   | ---
1. Modularity and Reusability         | 1. Steep Learning Curve
2. Separating Tests from Testbenches  | 2. Still in development
3. Simulator independent              | 3. Too many Functions and Tasks
4. Sequence based Stimulus generation | 4. Overkill for small agents
5. Configuration mechanisms           | 5. Bottleneck - lot of code to create TB
6. Factory mechanisms                 |
7. Plug and Play                      |
8. Bidirectional                      |
9. Phasing mechanisms                 |

Copy|Clone|Create
---|---|---
The copy makes this object a copy of the specified object.|The clone method creates and returns an exact copy of this object.|The create method allocates a new object of the same type as this object and returns it via a base uvm_object handle.
The do_copy method is the user-definable hook called by the copy method.|The default implementation calls create followed by copy.  As clone is virtual, derived classes may override this implementation if desired.| Pure virtual class. Hence, every class deriving from uvm_object, directly or indirectly, must implement the create method.

## uvm_object
| SEEDING         | IDENTIFICATION  | CREATION | PRINTING       | RECORDING
| ----            | ----            | ----     | ----           | ----
| use_uvm_seeding | set_name        | create   | print          | record
| reseed          | get_name        | clone    | sprint         | do_record
|                 | get_full_name   |          | do_print       |
|                 | get_inst_id     |          | convert2string |
|                 | get_inst_count  |          |                |
|                 | get_type        |          |                |
|                 | get_object_type |          |                |
|                 | get_type_name   |          |                |

| COPYING | COMPARING  | PACKING    | UNPACKING    | CONFIGURATION
| ----    | ----       | ----       | ----         | ----
| copy    | compare    | pack       | unpack       | set_int_local
| do_copy | do_compare | pack_bytes | unpack_bytes | set_string_local
|         |            | pack_ints  | unpack_ints  | set_object_local
|         |            | do_pack    | do_unpack    |

## Agent Mode
The int configuration parameter `is_active` is used to identify whether this agent should be acting in active or passive mode.  This parameter can be set by doing:
```
uvm_config_int::set(this, "<relative_path_to_agent>, "is_active", UVM_ACTIVE);
```
`get_is_active()` returns UVM_ACTIVE if the agent is acting as an active agent and UVM_PASSIVE if it is acting as a passive agent.
## Starting a test
A particular test case can be selected and execute on two methods,
1. by specifying the test name as an argument to run_test();
   * example: `run_test("mem_model_test");`
2. by providing the UVM_TESTNAME command line argument
   * example: `<SIMULATION_COMMANDS> +UVM_TESTNAME=mem_model_test`
- run_test( ) within tb_top as shown above, is a global task which is responsible for getting a reference to the uvm_root class instance from UVM core services.
- There is another run_test( ) method within uvm_root to
   - phases all components through all registered phases.
   - initialize factory settings
   - report servers
   - do basic level checks
## Pre and post body in a Sequence
The `*_body()` callbacks are designed to be skipped for child sequences, while `*_start()` callbacks are executed for all sequences.
```
virtual task start (
  uvm_sequencer_base sequencer, // Pointer to sequencer
  uvm_sequence_base parent_sequencer = null, // parent sequencer
  integer this_priority = 100, // Priority on the sequencer
  bit call_pre_post = 1 // pre_body and post_body called
);
```
## Seq arbitration
`seqr.set_arbitration();` Specifies the arbitration mode for the sequencer.
```
function void set_arbitration(
UVM_SEQ_ARB_TYPE val
)
```
Arbitration                | Order
---                        | ---
UVM_SEQ_ARB_FIFO (default) | Requests are granted in FIFO order
UVM_SEQ_ARB_WEIGHTED       | Requests are granted randomly by weight
UVM_SEQ_ARB_RANDOM         | Requests are granted randomly
UVM_SEQ_ARB_STRICT_FIFO    | Requests at highest priority granted in FIFO order
UVM_SEQ_ARB_STRICT_RANDOM  | Requests at highest priority granted in randomly
UVM_SEQ_ARB_USER           | Arbitration is delegated to the user-defined function, user_priority_arbitration.  That function will specify the next sequence to grant.

## Exclusive access for the seqr
*different from m and p seqr so need help
might be lock*

m_sequencer                                                                     | p_sequencer
---                                                                             | ---
generic uvm_sequencer pointer                                                   | typed-specific sequencer pointer
 initialized when the sequence is started                                       | you would need to typecast the m_sequencer to the physical sequencer
a handle of type uvm_sequencer_base which is available by default in a sequence | created by registering the sequence to the sequencer using macros.It will not exist if we have not registered the sequence with macros
## Default sequencer
- m_sequencer, available in any sequence, is the default sequencer
- A specific sequencer can be set using
  - `uvm_sequence_item::set_sequencer()` in sequence_item
  - `create_item( ..., .l_sequencer(seqr))` in sequence

## Sequencer - Driver Interactions
- `get_next_item` and `get` will retrieve the next available item from a sequence. The call will block until an item is available.
- `try_next_item` will try to retrieve if available, if not returns immediatly with request set to null object.
- `peek` will retrieve the *present availabe* item from a sequence. The call will block until an item is available.
  - Once a request item has been retrieved and is in the sequencer FIFO, subsequent calls to peek will return the same item.
  - The item will stay in the FIFO until either get or item_done is called.

The following steps occur on these calls:

No.|Retrieves the next available item from a sequence
---|---
1|Seq Arbitration
2|return from `wait_for_grant` in sequence
3|Randomizaion
4|Indicate `item_done` to the sequencer (only in `get`)
5|return with a reference to the item

`item_done` will :-
- Indicates that the request is completed to the sequencer.
  - Any uvm_sequence_base::wait_for_item_done calls made by a sequence for this item will return.
- The current item is removed from the sequencer FIFO.
- If a response item is provided, then it will be sent back to the requesting sequence.
  - The response item must have its sequence ID and transaction ID set correctly, using the uvm_sequence_item::set_id_info

## Factory
Using the factory involves three basic operations
1. Registering objects and components types with the factory
2. Designing components to use the factory to create objects or components
3. Configuring the factory with type and instance overrides, both within and outside components

new | create
---|---
SV|Factory - UVM
class constructor which is used to create an instance of a class|factory method that is used to create the instances of uvm components & objects with the help of class constructor
No Factory registeration| Factory can only create the instances of the class which are registered with it
No Factory Overriding| Can be overrided
