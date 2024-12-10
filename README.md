Hi Mohit/Atul,

While working on CBOPTIMIZER-11228, we identified an issue with the "First Name" field.

In the response from the resolve call, the first name is not being populated correctly. Instead of including the full first name, it only contains the first part if the name has a space. We observed the same behavior in the clientDetail call as well.

For example:

Full name: Veda Shruthi Poolla
First name is being populated as “Veda” instead of “Veda Shruthi”
The last name is populated correctly.
Attached are the relevant screenshots and the JSON response for your reference. Kindly investigate this issue and let me know if you need any further information.








