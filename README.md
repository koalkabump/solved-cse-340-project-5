Download Link: https://assignmentchef.com/product/solved-cse-340-project-5
<br>
<h2>CSE 340 Project 5</h2>

<h6 class="product-description">CSE 340 Project 5</h6>

5/5 - (7 votes)

Your goal is to write the front-end for a compiler. This front-end is responsible for parsing the input language and creating an intermediate representation. The back-end of the compiler takes in the intermediate representation and executes that intermediate representation. The back-end of the compiler is already written and cannot be changed.

Grammar Descriptionprogram → var_section bodyvar_section → id_list SEMICOLONid_list → ID COMMA id_list | IDbody → LBRACE stmt_list RBRACEstmt_list → stmt stmt_list | stmtstmt → assign_stmt | print_stmt | while_stmt | if_stmt | switch_stmtassign_stmt → ID EQUAL primary SEMICOLONassign_stmt → ID EQUAL expr SEMICOLONexpr → primary op primaryprimary → ID | NUMop → PLUS | MINUS | MULT | DIVprint_stmt → PRINT ID SEMICOLONwhile_stmt → WHILE condition bodyif_stmt → IF condition bodycondition → primary relop primaryrelop → GREATER | LESS | NOTEQUALswitch_stmt → SWITCH ID LBRACE case_list RBRACEswitch_stmt → SWITCH ID LBRACE case_list default_case RBRACEcase_list → case case_list | casecase → CASE NUM COLON bodydefault_case → DEFAULT COLON body

The tokens used in the grammar description are (note PRINT is not what you would expect):

SEMICOLON = ;ID = letter(letter | digit)*COMMA = ,LBRACE = {RBRACE = }EQUAL = =NUM = 0 | (digit digit*)PLUS = +MINUS = –MULT = *DIV = /PRINT = printWHILE = WHILEIF = IFGREATER =LESS = &lt;NOTEQUAL = &lt;SWITCH = SWITCHCASE = CASECOLON = :DEFAULT = DEFAULT

Lexingcompiler.h and compiler.c also define the lexing functions (getToken and ungetToken) which you should use for reading in the tokens.

Language SemanticsDivision is integer division (just as in C) and the result of the division of two integers is an integer.All variables must be explicitly declared in the var_section and have global scope.All variables are of type integer.All variables initially have the value 0.All statements in a statement list are executed sequentially according to the order in which they appear.Boolean ConditionA boolean condition takes two operands as parameters and returns a boolean value.

If StatementsAn if_stmt has the following (standard) semantics (note that our language does not have an else clause to the if_stmt):

The condition is evaluated.If the condition evaluates to true, then the body of the if_stmt is executed, then the next statement following the if_stmt is executed.If the condition evaluates to false, then the next statement following the if_stmt is executed.While Statementswhile_stmt has the following (standard) semantics:

The condition is evaluated.If the condition evaluates to true, the body of the while_stmt is executed, then goto step 1.If the condition evaluates to false, then the next statement following the while_stmt is executed.Switch Semanticsswitch_stmt has the following (standard) semantics:

The value of the switch variable is checked against each case number in order.If the value matches the case number, the body of the case is executed, then the next statement following the switch_stmt is executed.If the value does not match the case number, the next case is evaluated.If a default case is provided and the value does not match any of the case numbers, then the body of the default case is executed, then the next statement following the switch_stmt is executed.If there is no default case and the value does not match any of the case numbers, then the next statement following the switch_stmtis executed.Print Semanticsprint_stmt will print out the value of the variable (give as the ID) at the time of the execution of the print_stmt.Intermediate RepresentationThe intermediate representation is a graph that the compiler back-end knows how to interpret and execute. All intermediate representation data structures are defined in compiler.h, and you are not allowed to change compiler.h or compiler.c, which are posted on the submission site.



You should become very familiar with the intermediate representation data structures in compiler.h and their usage to execute the program in the execute_program function in compiler.c.

StatementNodeThe main data structure of the intermediate representation is the StatementNode, which has a type field which specifies what type of statement it is (AssignmentStatement, PrintStatement, IfStatement, or GotoStatement) and a field next which points to the next StatementNode in the list. Thus, StatementNode forms a linked list of statements that should be executed, in order. If the nextfield is NULL, then program execution terminates.

ValueNodeThe ValueNode data structure is used to represent a value in the program (both literals and variables). You should read compiler.cto understand how ValueNode is used when executing the intermediate representation.

AssignmentStatementThe intermediate language AssignmentStatement is constructed of a left_hand_side (the ValueNode that is being assigned to) and a potential operation performed on two operands. The op field specifies the type of the operation to perform, and an op value of OP_NOOP means that no operation is performed, so the value associated with operand1 is copied to the value associated with left_hand_side.

PrintStatementThe intermediate language PrintStatement has a ValueNode id as its only field, which specified the value to print out.

IfStatementThe intermediate language IfStatement is more complicated structure than the other structures. Similar to an AssignmentStatement, the condition of an IfStatement must be an operator, represented by condition_op. This operator is applied to condition_operand1 and condition_operand2, and if the result is true, then program execution continues executing the StatementNode in the true_branch, otherwise program execution continues executing the StatementNode in the false_branch.

GotoStatementThe intermediate language GotoStatement is a fairly simple construct. It has one field, target, that is the StatementNode to begin executing from when the GotoStatement is executed.

ImplementationYou are required to implement the function parse_generate_intermediate_representation (in another file than compiler.c and compiler.h, as you cannot modify these files) with the following signature:

struct StatementNode* parse_generate_intermediate_representation();

You can implement this function in C or C++, follow the guidelines in compiler.h.

You are free to write whatever helper functions you need (and really you should, a single huge function is not a good way to code), but you cannot modify compiler.h and compiler.c.

You can assume that all input programs will properly type check and that there will not be any syntax errors.

Hopefully you have noticed by now that there is a gap between the language semantics and the intermediate representation semantics. Essentially, your goal is to overcome this gap, by parsing the program and translating program semantics that do not exist in the intermediate representation semantics into semantically equivalent intermediate representation. This is the heart of the compiler, and it is exactly what GCC does when it compiles your C program to x86 assembly: x86 does not have while loops or switch statements, but C does.

Consider the program language if_stmt, which you must translate into a corresponding intermediate language IfStatement. We want the semantics of the if_stmt to be preserved when performing this translation.

One way to do this would be to create a StatementNode data structure, which has a type of IF_STMT. The if_stmt field of the StatementNode is an IfStatement. The true_branch should be the body of the if_stmt, and the false_branch can be a NOOP_STMT.

However, this is not enough! Due to the semantics of the intermediate language, if the IfStatement condition evaluates to false, then execution starts at the false_branch. This branch is a NOOP_STMT, so that is executed, then execution terminates. This does not follow the semantics of our original language! The program should continue execution at the next statement following the if_stmt. The same logic causes a problem if the IfStatement condition evaluates to true. The body will execution, then execution will terminate.

Therefore, we must ensure that the next field of the last StatementNode in the true_branch and the false_branch is the StatementNode that follows the if_stmt.

One way to implement this is in the following diagram:

A similar situation occurs when implementing the while_stmt, however the semantics of the while_stmt can be implemented using a GotoStatement as the last instruction of the true_branch, as the following diagram demonstrates:

switch_stmt must be handled similarly.

ExamplesYou will be given all the test cases for assignment statements, if statements, and while statements. However, here are some additional examples:

a;{a = 10 ;print a;a = 20;print a;}

Should output:

1020

The program:

a;{a = 10 ;SWITCH a{CASE 0 :{a = 20 ;}CASE 10 :{a = a + 40;}}print a ;}

Should output:

50

EvaluationYour submission will be graded on passing the automated test cases.

The test cases (there will be multiple test cases in each category, each with equal weight) will be broken down in the following way (out of 100 points):

Assignment statements : 20If statements : 25While statements : 25Switch statements : 20All statements : 10Note that all test cases depend on successful implementation of print statements (otherwise, how do we test the output), and if, while, and switch statement test cases all depend on assignment statements.

There is extra credit available (described in the following section):

For loops : 5Labels and GOTO : 5Just as in the normal assignment, you’re not allowed to change compiler.c or compiler.h to implement the extra credit.

Extra CreditExtend the language by supporting for loops and/or goto statements in the following way:

program → var_section bodyvar_section → id_list SEMICOLONid_list → ID COMMA id_list | IDbody → LBRACE stmt_list RBRACEstmt_list → stmt stmt_list | stmtstmt → assign_stmt | print_stmt | while_stmt | if_stmt | switch_stmt | for_stmt | label_stmt | goto_stmtfor_stmt → FOR assign_stmt condition SEMICOLON assign_stmt bodylabel_stmt → ID COLONgoto_stmt → GOTO ID SEMICOLONassign_stmt → ID EQUAL primary SEMICOLONassign_stmt → ID EQUAL expr SEMICOLONexpr → primary op primaryprimary → ID | NUMop → PLUS | MINUS | MULT | DIVprint_stmt → PRINT ID SEMICOLONwhile_stmt → WHILE condition bodyif_stmt → IF condition bodycondition → primary relop primaryrelop → GREATER | LESS | NOTEQUALswitch_stmt → SWITCH ID LBRACE case_list RBRACEswitch_stmt → SWITCH ID LBRACE case_list default_case RBRACEcase_list → case case_list | casecase → CASE NUM COLON bodydefault_case → DEFAULT COLON body

With the following added tokens:

FOR = FORGOTO = GOTO

For Loop SemanticsA for_stmt has the following (standard) semantics:

The first assign_stmt is executed.The condition is evaluated.If the condition evaluates to true, the body of the for_stmt is executed, then the second assign_stmt is executed, then goto step 2.If the condition evaluates to false, then the next statement following the for_stmt is executed.For example, the following program:

a;{FOR a = 10; a 0; a = a-1;{print a;}}

Will output the following:

10987654321

Goto SemanticsA goto_stmt has the following (standard) semantics:

The next execution after the goto_stmt will be the label_stmt that has an ID that matches the ID of the goto_stmt.A label_stmt is a NOOP.

For example, the following program:

foo;{foo = 1;label:IF foo &lt; 100{foo = foo + 1;IF foo 10{GOTO end;}GOTO label;}end:print foo;}

Should output:

11