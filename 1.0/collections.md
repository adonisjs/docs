# Collections

Collections attach methods to arrays and dictionaries so that you can manipulate them with ease. Under the hood `Lucid` also returns a copy of collection when you fetch values from your database.


## Grabbing Collection
\\`\`\`javascript,line-numbers
const Collection = use('Collection')
\\`\`\`


## Converting array to collection instance
\`\`\`javascript,line-numbers
var users = [
  {
	username: 'foo',
	age: 22
  },
  {
	username: 'bar',
	age: 25
  },
  {
	username: 'baz',
	age: 22
  }
]

const usersCollection = new Collection(users)

\`\`\`

Now you can make use of different methods on top of your collection object.

#### sortBy

\`\`\`javascript,line-numbers
const sortedUsers = usersCollection.sortBy('age').toJSON()
\`\`\`

In fact, you can make use of any [lodash][1] method, as `Collection` returns wrapped instance of lodash.

[1]:	https://lodash.com/docs#chain