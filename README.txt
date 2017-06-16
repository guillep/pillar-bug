Install stable Pillar version

$ git clone https://github.com/pillar-markup/ToHack
$ cd ToHack
$ ./download.sh

Open Pharo

$ ./pharo Pharo.image test "Pillar.*"
[...]
3182 run, 3182 passes, 0 failures, 0 errors.

Introduce in PRPillarConfiguration a disabledPhases instance variable + accessors
(You can do it with the following script)

$ ./pharo Pharo.image ../introduceBug.st --save

Run tests again:

$ ./pharo Pharo.image test "Pillar.*"
[...]
3182 run, 3166 passes, 0 failures, 16 errors


The affected tests are failing because now, in the method PREPubMenuJustHeaderTransformer>>actionOn:

```smalltalk
actionOn: anInput
	^ (self class writers
		includes: anInput configuration outputType writerName)
		ifTrue: [ maxHeader := self maxHeaderOf: anInput input.
			super actionOn: anInput ]
		ifFalse: [ anInput ]
```

The error is caused because outputType is nil. 


Alternatives to reproduce the issue:

PRExportBuilder new
	createConfiguration: 'pillar.conf' 
	baseDirectory:  FileSystem workingDirectory   
	argDictionary: {
		'inputFile'-> (FileSystem workingDirectory / 'Chapters/Chapter1/chapter1.pillar') .
		'defaultExporters' -> {'latex'}
		} asDictionary;
	export

Alternative Semantics

$ ./pharo Pharo.image ../introduceBugAlternative.st --save
[...]
3182 run, 3175 passes, 0 failures, 7 errors.