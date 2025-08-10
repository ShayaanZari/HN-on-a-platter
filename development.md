## Node I/O

n8n expects the following structure as node I/O:
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
