# Web interface

DNSvizor comes with a web interface served over https.

## Dashboard

The landing page of the DNSvizor web interface is the *dashboard*.
The dashboard displays metrics about the DNS queries, DNS cache size, DNS block lists and memory usage.

## Block list

The block list page displays information about blocked domains.
At the top are the domains "manually" blocked either through boot arguments (see [DNSvizor options](./dnsvizor_options.md)) or via the web interface.
Each blocked domain can be unblocked by the "delete" button.
Below that is another list of remote block lists.
There you find what block lists are used as well as the number of domains blocked through that block list source.
Finally, at the top you find a text input field where you can type in domains you want to block.
Next to the input field is a refresh button ("⟳") that can be clicked to refresh the remote domain block lists.
This operation may take some time.


## Configuration

**TODO**: configuration page.
