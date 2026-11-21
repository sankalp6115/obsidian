```bash
fm chat
fm respond "query"
fm respond "query about image" --image "image.png"
fm schema object --name "fieldName" --string "anotherField" --array > schema.json
fm respond "query" --schema schema.json
```

#### An example on how you can do computer tasks using the model.

```bash
fm schema object --name "FilteredFiles" --string "final_files" --array --string "draft_files" --array > schema.json
files_list="${ls -1 $(DIRECTORY_TO_TRIAGE)}"

output=$(fm response
  --instructions "i just completed a project, and I need help triaging the latest version of the file from the previous versions. I will give you a list of files. Return a list of the latest files (i.e., all files that, you can infer from their name in the list, are the latest versions), and then return separately a list of all draft files (i.e., all files that weren't considered final.)" \
  "This is the list of all files:\n\n${files_list}")
  --schema schema.json 
```

![[Pasted image 20261111020819.png]]


### Python SDK
```sh
pip install apple_fm_sdk
```

- Tool Calling
- Text and Image input
- Streaming
- Guided Generation 

![[Pasted image 20261111021054.png]]
![[Pasted image 20261111021109.png]]