While twitt from Uncle Bob is booming, I am wondering are there all metrics, indicators in place, so we are ok to stop reviewing generated code.  
As code generation considered to be solved a problem, we can now generate code fast and cheap. More and more the opinion starts to prevail, that next bottleneck in system engineer is verification. I believe there is still a room for improvements. Can we deliver to fearlessly critical systems to production purely looking at few indicitors? What these indicators should be?  

- Branch/code coverage give only partial view that code was exercised. It does not give information whethere tests have good asserts that corresponds to business expecttions. 
- Mutation tests that tests help with to detect missing coverage for the edge cases. should be there something more? 
- Negative tests. How to understand that we check that system does not do something additional like sending credentials over the wire somhere.
Do we need more metrics, indicators? 

- Is there a metric that says how system is flexible for changes? and we should not regenerate everything on requirements change. One can argue that regeneratio is cheap now, but still imposes risk that after regeneration we introduce bug, missing tests. Mechanics still remain the same. Generation should remain under control. 
- Another bit on tests and assertions. Idea about how we sync our expectations with coding agents. When we develop a product do we already know upfront all edge cases which can occur. Even we have not thought about the situation in which product can appear, can we be sure that agent will find a good way to solve this, that is algined without our and business vision of the world. 

For not so sensitive or critical systems piloting and deployment modes with rapid roll back on failure could help, but it is not the case for everything.  

Looking forward if this field evolve more.



