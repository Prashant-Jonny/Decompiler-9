# Decompiler
Decompiler .smx to .sp for Sourcemod.
D'après l'idée original de "BAILOPAN" https://forums.alliedmods.net/showthread.php?t=170898 pour le décompiler Lysis.
Java/C#

Remarques :
- Lisibilité du code dépend du code en question et du nombre de lignes.
- Recompilation après la décompilation impossible ou défectueuse. 

Technical Details

The decompiler starts by decompressing and extracting each section of the .smx file. These are sections like the list of exported functions, and the compiled instruction stream. The instruction stream is then transformed into a low-level IR (LIR) on a per-function basis. Jump instructions and their targets are used to compute a control-flow graph. A number of analyses then take place on the CFG: a dominator tree is computed, and the boundaries and nesting of loops is computed.

Pawn is basically a stack machine and LIR is pretty inconvenient. Originally Lysis went from LIR to an expression tree, but expression trees are difficult to analyze. Now, we transform LIR to an SSA-form IR. This IR has a lot more information, like an embedded dataflow graph. For example, it is trivial to find all IR nodes that use another IR node, and thus it is trivial to rewrite the graph as we discover new information.

Pawn is really low-level, so even with SSA, it is pretty tricky to analyze. Some examples:
Floating-point operations are implemented as a stock that calls a native. We essentially pattern-match calls to these stocks and rewrite the call to be a comparison.
Array operations work by computing references. These references can be computed in a number of ways. For example, a[x] can be computed with "add a, x". When loading and storing to references, we have to pattern match sequences like this to compose a proper array+index pair.
The instruction stream has no type information, and often passes around random addresses as if they were normal integers/cells. Dealing with this is hard. We use two type propagation passes. The first is forward, and propagates information such as "a comparison is boolean" and "loads and stores must operate on references". The second pass is backward, and does things like taking a function call, and propagating the types of the call's signature to its given operands.
The instruction stream has no concept of scope, so the way loops are decomposed can generate duplicate variable names.
I have not yet found a nice way to reconstruct a pretty "for" loop, so they appear as "while" loops.
The compiler generates || and && as a really nasty chain of implicit "if"s. There is a really complicated pattern-matching algorithm to track these down and recompose the original expression.
Currently, Lysis has difficulty figuring out x[a][b] where "x" is a global and "a" and "b" are constants. I hand-edited func_37 since I was running out of time.

The original goal of the decompiler was to continue working even in the presence of highly obfuscated binaries. By now, it's reached a tradeoff where it does a lot of pattern matching but also uses flexible analysis phases. However, there are still significant challenges I haven't figured out (and probably won't). Hopefully though, it has enough interesting stuff to be, at least, of educational value.
