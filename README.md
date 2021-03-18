## How not to Autogenerate Passwords
I hope that everyone generates passwords automatically these days, and has them stored in some kind of password manager. In this short article, I would like to discuss how not to shoot yourselves in the foot in the process.

### Password Generator Options
I don't want to dive deep in the cryptography waters, partly because I might drown and partly because that is a different topic. Let's stay practical. The password generators usually offer you a pick in the password length, some character classes, and sometimes whether or not you want to generate the password on your device.

- Starting from the last, make sure that the password generator doesn't send data over the internet (e.g. by checking network in your browser's developer tools)
- Any password length below 8 characters is brute-forcable. 8 is already impossible. I pick 16 and don't give it a second thought unless some site actually forces me to pick shorter, which always brings me to question sanity of the process that produced it
- Lastly, character classes. The more the merrier, right? Wrong!

### The hidden evil lurking in character classes
Let's see which character classes we can have first ant talk about them later, but first let me tell you what these somewhat artificial classes are and are not. These are not character classes from the *regex* perspective, rather classes which you might encounter in password generators, which is where I've actually taken it from.

Class                | Example
---------------------|-----------------------
Numbers              | `1,2,3,...`
Lowercase letters    | `a,b,c,...`
Uppercase letters    | `A,B,C,...`
Symbols              | `@,#,%,^,_,-,$,!,&,*`
Similar characters   | `i,I,1,l,L,o,O,0,...`
Ambiguous characters | `{,},[,],',",:,...`

*Numbers* and *letters*, there is no question in there. If you are typing the password, you might want to omit *Similar* characters, even if you are using a font that makes clear distinctions between, for example, `0`(zero) and `O`(capital O). Anyway, these are not dangerous despite being a nuisance sometimes.

Now, *Similar characters* and *Ambiguous characters*, that's a different story. 

Imagine you need to use the password in a server or an application. It might go into a configuration file, be stored in memory as environment variable, or given as a startup paramater for a process, for example a database connection string. Let's stick with one example. You're starting up a server, which is bootstrapping with a shell script, and you need to substitute a `PLACEHOLDER` with a `pas&word` in a configuration file.

That's simple, you just substitute it with `sed`:
```zsh
echo "property.pass=PLACEHOLDER" | sed 's/PLACEHOLDER/pas&word/'
property.pass=pasPLACEHOLDERword
```
...oh. It turns out that the ampersand inserts the whole matched sequence. You will encounter similar problems with `$` in `awk` or plain shell. Thinking about escaping this character just kicks the can of worms down the street and opening it more. But you can say that it is in single quotes (which are also evil), and you just need to throw away `&`. Right, but you don't have password in `sed` just like that, do you. You need to pull it from somewhere, for example `aws secretsmanager`. So you need to lose the single quotes:
```zsh
SECRET=password
echo "property.pass=PLACEHOLDER" | sed 's/PLACEHOLDER/'"$SECRET"'/'
property.pass=password
```
That is of course sensitive to internals of your shell, but at least now you have a way how to render your template with secret without having it all over your server.
I think the example is quite convincing, but if it is not, please go on and waste your life.:) I've learned this lesson already. Let me just finish with the list of unsafe symbols I've found, and which you should not use in your passwords:

- `!` ... shell internal (event)
- `*` ... shell globbing
- `$` ... shell almost everything
- `&` ... sed insert matching sequence
- `\` ... escape sequence everywhere
- `/` ... sed divider (usually)
- `'` ... end/finish string everywhere
- `"` ... ditto

If there are any other evil characters, please let me know!
