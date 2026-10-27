

# Oracles

Traditionally in vulnerability research or exploit development fields, an **oracle** is anything that reveals information about something you don't have access to. Oftentimes, a model does not have domain-specific knowledge to accurately answer research questions we have when conducting vulnerability research. To get better results out of our vulnerability research, we need to expose an interface, an oracle, to our harness that has data the model requires to answer our research questions. An oracle doesn't make security decisions on its own. Instead, it provides data that humans or agents can use to make more informed decisions.


# Case study: Nodes/Proxy RCE
Let's take a look at an oracle I built when disclosing a remote code execution vulnerability in Kubernetes and why I needed to build an oracle. In Kubernetes, permissions are generally granted using role-based access control (RBAC). I had identified a quirk in the permissions system and needed to identify what vendors utilized this RBAC permission so I could understand the blast radius. Asking an agent to get this data is unreliable. Simply using a tool call to search the web didn't yield useful or comprehensive results.

To better answer this question, an oracle was needed. Specifically, I needed an oracle that pulled in the data I needed to answer this question. The result of this was a web application that pulled charts from Helm repositories, unpacked their templates and values, extracted their RBAC declarations, and stored the permissions in a searchable database. Using this, I could filter across the database to find the information I needed.

This changed the task from an open-ended question for an agent into a reproducible query. Instead of asking an agent to search the web and tell me which vendors might be affected, it allowed me to do actual data analysis on:

- The chart and version
- The declared resource and verbs
- Whether the permission was namespace- or cluster-scoped
- The Role or ClusterRole that declared it
- And more
