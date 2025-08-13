Torq Loops – Detailed Notes
Loop Basics

    Loop Types:

        Sequential – Executes one iteration at a time (supports break).

        Parallel – Executes iterations concurrently (no break support).

    Naming Loop Variables:

        Give meaningful names for loop keys & values to keep data handling clear.

        Example:

            generate_key / generate_value for the outer loop.

            num_key / num_value for inner loops.

Generating Data with Loops

    Creating an Array

        Use a loop to generate fixed iterations (e.g., 5 elements, step size 1).

        Can populate with random numbers in each iteration:

    {
      "number": <RANDOM_INT_1_TO_6>
    }

Collecting Loop Output

    Use Collect operator to aggregate loop results into a single array in context.

    Example result:

        [
          {"number": 4},
          {"number": 2},
          {"number": 5}
        ]

    Summing Loop Results

        Use a context variable (total) and update it in each iteration:

            Start with total = 0.

            Add current number to total.

        Produces cumulative sum.

Nested Loops Example

    Outer loop – Iterate over random numbers.

    Inner loop – For each number, generate random strings.

    Collected output – JSON object pairing number with generated strings.

Example:

[
  {
    "number": 4,
    "strings": ["abc", "xyz"]
  },
  {
    "number": 2,
    "strings": ["pqr", "lmn"]
  }
]

Breaking a Loop

    Use Break operator when a specific condition is met.

    Supported only in Sequential loops.

    Example:

        break if number == 3.

Collect Operator Nuances

    Collect only includes values generated since last execution in that loop.

    Nested loops each have independent collect buffers.

    Use Final Results operator for:

        Pagination results

        Merging multiple pages into a single array.

Loops Cross-Platform Cheatsheet
Feature / Concept	Torq	Shuffle	XSOAR (Palo Alto)	Splunk SOAR
Loop Types	Sequential, Parallel	Sequential, Parallel	For Each (sequential), parallel playbooks	Loop blocks & playbook for-each
Break Support	Sequential only	Sequential only	Manual break using conditions	Manual break logic in decision blocks
Data Source	Array, JSON	Array, JSON	Context list, automation script output	Container artifacts, list data
Collect Results	Collect operator	Accumulate in workflow variable	Append to context key	Playbook list variable
Nested Loops	Yes, with named keys/values	Yes	Yes	Yes
Sum / Count Operations	Context variables	Stored variables or Python script step	Built-in transformer functions	Built-in function or custom function
Final Results	Final Results operator	End-of-loop accumulator	Last task aggregation	Custom “merge list” block
Practice Lab – Shuffle Version of Torq Loops

Goal: Replicate Torq’s “Loops” training in Shuffle to gain hands-on skill without Torq access.

    Outer Loop – Generate Numbers

        Input: [1, 2, 3, 4, 5]

        Action: Assign each iteration a random int (1–6).

        Store in variable random_numbers.

    Sum Random Numbers

        Create variable total = 0.

        Add each random number to total inside loop.

    Inner Loop – Generate Strings

        For each number in random_numbers, generate N random strings where N = number value.

        Store each { "number": X, "strings": [...] } in a results array.

    Break Condition

        If number == 3, break outer loop.

    Final Output

        Output results array to verify structure.