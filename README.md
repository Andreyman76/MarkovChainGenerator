# Markov Chain Generator
Adds the *MarkovGenerator* class, which allows you to generate different subsets of elements using Markov chains.
## Usage:
The order of the elements of the original sets matters, since the generator remembers which elements can be the beginning or end of the set.

To generate new random subsets, you need to create a generator object, initialize it with the data of the original set and call *generate* method.
```python
from markov_generator import MarkovGenerator


generator = MarkovGenerator()
generator.add_sentence([1, 2, 3])
generator.add_sentence([3, 4, 5])

for i in range(5):
    print(generator.generate())
```
### Output:
```
[1, 2, 3, 4, 5]
[3, 4, 5]
[1, 2, 3, 4, 5]
[3, 4, 5]
[1, 2, 3]
```
The *print_nodes* method prints the generator nodes to the console. Nodes **[START]** and **[END]** indicate the beginning and end of sentences.
```python
from markov_generator import MarkovGenerator


generator = MarkovGenerator()
generator.add_sentence([1, 2, 3])
generator.add_sentence([3, 4, 5])

generator.print_nodes()
```
### Output:
```
[START]: {sum: 2, postfixes: {(1,): 1, (3,): 1}}
(1,): {sum: 1, postfixes: {(2,): 1}}
(2,): {sum: 1, postfixes: {(3,): 1}}
(3,): {sum: 2, postfixes: {'[END]': 1, (4,): 1}}
(4,): {sum: 1, postfixes: {(5,): 1}}
(5,): {sum: 1, postfixes: {'[END]': 1}}
```
You can also use the *most_likely_continuation* method to select the most likely continuation (except **[END]**) of a given lexeme. Returns **None** if no continuation was found for the given lexeme.
```python
from markov_generator import MarkovGenerator


generator = MarkovGenerator()

generator.add_sentence('hello Jack'.split(' '))
generator.add_sentence('hello Mike'.split(' '))
generator.add_sentence('hello Sam'.split(' '))
generator.add_sentence('hello Jack'.split(' '))

continuation = generator.most_likely_continuation('hello')
print(''.join(continuation))
```
### Output:
```
Jack
```
### More examples in examples.py
## Additional functions in examples.py:
The *get_syllables* function allows you to get all the syllables from a word (not necessarily grammatically correct, but readable).
```python
from examples import get_syllables

syllables = get_syllables('pneumonoultramicroscopicsilicovolcanoconiosis')
print(syllables)
```
### Output:
```
['pne', 'u', 'mo', 'no', 'ult', 'ra', 'mic', 'ros', 'co', 'pic', 'si', 'li', 'co', 'vol', 'ca', 'no', 'co', 'ni', 'o', 'sis']
```

The *group_by* function allows you to split a set into groups of a given maximum length.

```python
from examples import group_by

groups_set = group_by([1, 2, 3, 4, 5, 6], 3)
for groups in groups_set:
    print(groups)
```
### Output:
```
((1, 2, 3), (4, 5, 6))
((1,), (2, 3, 4), (5, 6))
((1, 2), (3, 4, 5), (6,))
```
<p>Grouping allows Markov chains to establish correspondence not only between two elements, but also between sets of elements.</p>
<p>This allows you to generate sets more similar to the original.</p>

```python
from markov_generator import MarkovGenerator
from examples import group_by

elements = [1, 2, 3, 1, 2, 4, 1, 2, 5]
print(f'Elements: {elements}')
groups_set = group_by(elements, 2)
generator1 = MarkovGenerator()
generator1.add_sentence(elements)  # Add a set without grouping
generator2 = MarkovGenerator()

print('\nGroups:')
for groups in groups_set:
    print(groups)
    generator2.add_sentence(groups)  # Add grouped set

print('\nGenerated without grouping:')
for i in range(5):
    print(generator1.generate())

print('\nGenerated with grouping:')
for i in range(5):
    print(generator2.generate())
```

### Output:
```
Elements: [1, 2, 3, 1, 2, 4, 1, 2, 5]

Groups:
((1, 2), (3, 1), (2, 4), (1, 2), (5,))
((1,), (2, 3), (1, 2), (4, 1), (2, 5))

Generated without grouping:
[1, 2, 3, 1, 2, 5]
[1, 2, 4, 1, 2, 5]
[1, 2, 4, 1, 2, 3, 1, 2, 3, 1, 2, 4, 1, 2, 4, 1, 2, 4, 1, 2, 3, 1, 2, 5]
[1, 2, 5]
[1, 2, 3, 1, 2, 4, 1, 2, 4, 1, 2, 4, 1, 2, 5]

Generated with grouping:
[1, 2, 3, 1, 2, 4, 1, 2, 4, 1, 2, 5]
[1, 2, 4, 1, 2, 5]
[1, 2, 4, 1, 2, 5]
[1, 2, 4, 1, 2, 5]
[1, 2, 5]
```
### Example of generating 5 unique fantasy names

```python
from markov_generator import MarkovGenerator
from examples import get_syllables


generator = MarkovGenerator()

# Loading the Generator with Examples
with open('FantasyMaleNames.txt', 'r', encoding='utf-8') as file:
    names = [name.replace('\n', '') for name in file.readlines()]
    
for name in names:
    syllables = get_syllables(name)  # Got a list of syllables of a name
    generator.add_sentence(syllables)  # Adding syllables to generator as sentence

# Generating 5 unique names
count = len(names)
while len(names) < count + 5:
    name = ''.join(generator.generate())
    if name not in names:  # Uniqueness check
        print(name)
        names.append(name)
```
### Output:
```
Xenakin
Amonix
Alexa
Dunixios
Viserian
```
### Example of generating 5 sentences

```python
from markov_generator import MarkovGenerator
from examples import group_by


with open('HarryPotterText.txt', 'r', encoding='utf-8') as file:
    text = file.read()

replaces = {
    '!': '.',
    '?': '.',
    '...': '.',
    ';': '',
    '”': '',
    '“': '',
    ':': '',
    '\n': '',
    '\r': '',
    ', ': ' ',
    '(': '',
    ')': ''
}

for key, value in replaces.items():
    text = text.replace(key, value)

sentences = text.split('.')
generator = MarkovGenerator()

for sentence in sentences:
    words = [word.lower() for word in sentence.split(' ')]
    while '' in words:
        words.remove('')

    generator.add_sentence(words)  # Add words to the generator as sentences
    groups_set = group_by(words, 2)
    for groups in groups_set:
        generator.add_sentence(groups)  # Add word groups to the generator as sentences

# Generate 5 random sentences
for i in range(5):
    print(' '.join(generator.generate()))
```
### Output:
```
they wouldn't be magic yet—what on the room was in his scar keeps getting to the last on harry with great muggle clothes
what are you three doing inside
he could take him
if you're worth knowing he hurried out from behind his paper
harry's heart
```