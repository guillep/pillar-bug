# Pillar-Bug

This repository explains how to reproduce a hard-to debug pillar bug due to the strange semantics of Pillar configurations.
The configuration of a pillar project uses `doesNotUnderstand:` and the Magritte meta-description framework to control how a configuration should be serialized and deserialized into text among others. This combination has strange semantics that produce hard-to-debug errors.. 

## Setting up the Bug Environment

Clone this repository

```bash
$ git clone https://github.com/guillep/pillar-bug.git
cd pillar-bug
```

Install stable Pillar version

```bash
$ git clone https://github.com/pillar-markup/pillar
$ cd pillar
$ ./download.sh
````

Run Pharo tests to see that all tests are running OK:

```bash
$ ./pharo Pharo.image test "Pillar.*"
[...]
3182 run, 3182 passes, 0 failures, 0 errors.
```

## Bug Alternative 1

### Introducing the Bug

Introduce in PRPillarConfiguration a disabledPhases instance variable + accessors
(You can do it with the script in this repository)

```bash
$ ./pharo Pharo.image ../introduceBug.st --save --quit
```

Run tests again and see that this accessor produced 16 errors:

```bash
$ ./pharo Pharo.image test "Pillar.*"
[...]
3182 run, 3166 passes, 0 failures, 16 errors
```

### Bug Symptoms

The affected tests are failing because now, in the method `PREPubMenuJustHeaderTransformer>>actionOn:`

```smalltalk
actionOn: anInput
	^ (self class writers
		includes: anInput configuration outputType writerName)
		ifTrue: [ maxHeader := self maxHeaderOf: anInput input.
			super actionOn: anInput ]
		ifFalse: [ anInput ]
```

The error is caused because outputType is nil. But we have no clue about the relation with the change we did and the bug.

### Alternatively, to reproduce from the image

Alternatives to reproduce the issue:

```smalltalk
PRExportBuilder new
	createConfiguration: 'pillar.conf' 
	baseDirectory:  FileSystem workingDirectory   
	argDictionary: {
		'inputFile'-> (FileSystem workingDirectory / 'Chapters/Chapter1/chapter1.pillar') .
		'defaultExporters' -> {'latex'}
		} asDictionary;
	export
```

## Bug Alternative 2

### Introducing the Bug

We can introduce the bug using a different version of the `disabledPhases` accessors, using the properties mechanism in the configuration.

```bash
$ ./pharo Pharo.image ../introduceBugAlternative.st --save --quit
```

Run tests again and see that this accessor produced 7 errors:

```bash
$ ./pharo Pharo.image test "Pillar.*"
[...]
3182 run, 3175 passes, 0 failures, 7 errors.
```

### Bug Symptoms

The affected tests are failing because now, in the method `PRCreationPhase class>>isEnabled:`

```smalltalk
PRCreationPhase class>>isEnabled: aConfiguration
	^ (aConfiguration disabledPhases includes: self key) not
```
