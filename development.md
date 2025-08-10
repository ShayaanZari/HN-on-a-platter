## Node I/O

### The Problem

1. n8n expects the following structure as node I/O

```JS
return [
	{
		json: {
			apple: 'beets', // the actual data is placed here
		}
		// Optionally, pairedItem can be an object key too
	}
];
```

2. While it is good practice to return the above, simply returning without the outer `json` key also works as n8n will wrap it automatically, as long as you are returning a JSON object.


3. The first HTTP Request node in the workflow sends a GET request for the 500 IDs and gets a JSON array of integer primitives:

```
[ 44850913, 44846281, 44850681, 44851214, 44851590, 44846922, 44851557, 44848293, 44849129, ..., 44824088 ]
```

Before sending it as input to the next node, n8n wraps each primitive as its own object:

```JS
[
	{
		json: 44850913
	},
	{
		json: 44846281
	}
	...
];
```

4. The node which needs the IDs is the SQLite filter node, which needs the IDs in a comma-separated list.

### Solution

To my knowledge, only Code nodes can take improper input. We already need one to slice the 30 top IDs, so I placed the formatting code in the same node. I returned a single JSON object containing a single String primitive holding the IDs as a comma-separated list.

```JS
// slice first 30 (frontpage)
let ids = $input.all().slice(0,30)

// list comprehension containing each id, joined into a string. 
let arr = ids.map(item => item.json).join(',')

// returning a json object containing a string
return [{
  id_list: arr
}];
```

If it were a typical node (not SQLite) taking this input, I would simply return an array of objects each containing an ID:

```JS
let ids = $input.all().slice(0,30)
return ids.map(item => ({ id: item.json} ));
```

## SQLite ID Filter
After the 30 IDs are passed properly, next is to filter out previously processed IDs. I do this by creating a temporary table. 
- The `json_each()` function is a table-valued function that returns a set of rows, each representing an element in the input JSON object/array.
- We create a temporary table by using `json_each()` on the id list after converting it into a JSON array (and then a string)
Select all values (call the values ID to maintain consistency in n8n) from the temporary table created from the input list of IDs, that are not values in the database.

```SQL
SELECT value AS id
FROM json_each('[{{ $json.id_list }}]')
WHERE value NOT IN (
    SELECT id FROM seen_ids
);
```

Also note that for n8n (self-hosted on laptop / later Raspberry Pi) to interact with a database (on device), the database must be placed in a mapped Docker volume, and the `node` user of the Docker container must be given read and write access to the directory containing the database.

## Classification
Experiments showed that the Agent/LLM Chain nodes won't append an output of "true" or "false" to the input JSON. It is a clear waste of tokens to give all the metadata in the prompt. So instead, I gave only the title to the LLM and then used the Merge node to put it together with the corresponding metadata.

<p align=center>
<img width="673" height="357" alt="Screenshot from 2025-08-09 11-47-47" src="https://github.com/user-attachments/assets/18e46ba7-0744-4a3f-b5c0-1a41f0b2bc52" />
</p>
