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

### Flow
Experiments showed that the Agent/LLM Chain nodes won't append an output of "true" or "false" to the input JSON. It is a clear waste of tokens to give all the metadata in the prompt. So instead, I gave only the title to the LLM and then used the Merge node to put it together with the corresponding metadata.

<p align=center>
<img width="673" height="357" alt="Screenshot from 2025-08-09 11-47-47" src="https://github.com/user-attachments/assets/18e46ba7-0744-4a3f-b5c0-1a41f0b2bc52" />
</p>

I utilized several new pieces of knowledge at this step:
- The LLM Chain node is a simpler version of the Agent. It has no tools and, according to ChatGPT, uses fewer tokens as a result.
- Splitting data into two equivalent streams. It can't be done with the + button, but by grabbing the circle at the edge of the node.
- The Merge node can utilize n8n's consistent item ordering to combine by position.
	- Originally, I was trying to pass along the ID into the prompt in order to index via ID later. The Merge node could probably do that too, but the LLM couldn't return ID as a key since everything it returns is embedded in the `text` key, requiring string parsing.

### Prompt
A straightforward few-shot classification prompt.

```
You are an expert material curator specializing in STEM. Your task is to classify titles that are related to the fields of math, physics, algorithms, algorithmic aspects of AI, or robotics. You must respond with only one of two possible classifications: "True" if the title is related to the above fields, or "False" if not.

Input: "A new proof for Fermat's Last Theorem"
Output: True

Input: "Simulating Quantum Entanglement with a Classical Computer"
Output: True

Input: "Designing an AI Model to Calculate Optimal Traffic Light Timings"
Output: True

Input: "YC's 2025 batch"
Output: False

Input: "Writing a storage engine for Postgres"
Output: False

Input: "How can ChatGPT serve 700M users when I can't run one GPT-4 locally?"
Output: False

Input: Title {{ $json.title }}
Output:
```

## Updating the Database

1. Redefinition of Schema

In the original design, only **interested**, **not-published-before** articles reach the end of the workflow where they inserted to the DB, Uninterested, not-published-before articles will be have to be reclassified as uninterested.

The clear solution to this is to redefine the database to hold all classified IDs. That is, insert all IDs into the database after classification so no classified ID will pass through the filter node.

2. Insertion of IDs

This is done right after the merge, when the IDs that have been classified have been met with their classifications. It could happen right after retrieving the info, or right after filtering, but it's a more robust design choice to do it after they've actually been classified. Though, I haven't added any error handling at this point.

## Discord Publishing
I had already made a Discord bot in an old project, and the actual Discord setup in n8n was very straightforward. The only hitch I encountered was that it wanted the timestamp in ISO 8601 format, while the HN API gave it in Unix/Epoch. Some googling revealed the solution to this:
```JS
{{ new Date($json.time * 1000).toISOString() }}
```
