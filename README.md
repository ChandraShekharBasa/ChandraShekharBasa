Hi Mohit/Atul,

We observed the issue regarding First Name while working on CBOPTIMIZER-11228.

In the resolve call response, we observed that first name is populated wrongly i.e. it does not contain the full first name instead only the first part of the name if the first name contains the SPACE.
Observed the same issue with clientDetail call as well.
For example: 

Full name: Veda Shruthi Poolla
First name is populated as “Veda” only instead “Veda Shruthi”
Last name is populated correctly

Please find the following screenshots & attached Json response for your reference.
Request you to look into this issue and let me know if you need any further details.

 


 


Thanks,
Chandra








