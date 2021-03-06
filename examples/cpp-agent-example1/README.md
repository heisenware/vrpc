# Example 1 - "Foo'ed again"

This is a very simple example demonstrating the basic steps needed to
make existing C++ code remotely callable.

> **NOTE**
>
>
> In order to follow this example from scratch, first download
> the correct C++ agent for your platform from [vrpc.io](https://vrpc.io/download).
>
> Save the tarball in a new directory (e.g.
> `vrpc-cpp-agent-example1`), and unpack it using:
>
> ```bash
> tar -xzf vrpc-cpp-agent-<platform>.tar.gz
> ```
>
> then rename the resultant directory to `third_party`, i.e.
>
> ```bash
> mv <platform> third_party
> ```
>
> Finally create a directory `src` and you are good to go.

## STEP 1: C++ code that should be bound

We pretend that the code below already existed and should be made remotely
accessible.

*src/Foo.hpp*

```cpp
class Foo {

  int _value = 0;

public:

  Foo() = default;

  int getValue() const {
    return _value;
  }

  void setValue(int value) {
    _value = value;
  }
};
```

## STEP 2: Main File

We are going to produce an executable that starts an agent and sits waiting
until it receives remote requests to call functions. Hence, we have to provide
a `main.cpp` file.

*src/main.cpp*

```cpp
#include <vrpc.hpp>
#include <vrpc_agent.hpp>

#include "Foo.hpp"

// Bind functions to be remotely callable
namespace vrpc {

  // Bind constructor
  // Needs: class, [arg1-type, [arg2-type...]]
  VRPC_CTOR(Foo);

  // Bind const member function
  // Needs: class, return-type, function, [arg1-type, [arg2-type...]]
  VRPC_CONST_MEMBER_FUNCTION(Foo, int, getValue)

  // Bind non-const member function
  // Needs: class, return-type, function, [arg1-type, [arg2-type...]]
  VRPC_MEMBER_FUNCTION(Foo, void, setValue, int)
}

int main(int argc, char** argv) {
  auto agent = vrpc::VrpcAgent::from_commandline(argc, argv);
  if (agent) agent->serve();
  return EXIT_SUCCESS;
}
```

Making functions remotely available takes nothing more than filling in some
macros. See the reference documentation for all details.

**IMPORTANT**: Always define the macros in the `vrpc` namespace.

## STEP 3: Compilation

Certainly, you want to use the builtin tools of your IDE or any other
make file generator (such as CMAKE, node-gyp, etc.) you feel familiar with.

You are encouraged to do so! For getting VRPC compiled only make sure you:

- add `third_party/include` to your include folders (`-I` compiler flag)
- add `third_party/lib` to your library folders (`-L` compiler flag)
- use `libvrpc_agent.a` as dependent library for a static build (`-l:` compiler flag)
- or use `vrpc_agent` as dependent library for a dynamic build (`-l` compiler flag)

For getting this example working we quickly write a Makefile by hand, like so:

*Makefile*

```Makefile
TARGET = vrpc-foo-agent
CPPFLAGS = -I./third_party/include -pthread -fPIC -m64 -O3 -std=c++14
LDFLAGS = -pthread -L./third_party/lib
LDLIBS = -l:libvrpc_agent.a

SRCS := $(shell find ./src -name *.cpp)
OBJS := $(addsuffix .o,$(basename $(SRCS)))
DEPS := $(OBJS:.o=.d)

$(TARGET): $(OBJS)
	$(CXX) $(LDFLAGS) $(OBJS) -o $@ $(LOADLIBES) $(LDLIBS)

.PHONY: clean
clean:
	$(RM) $(TARGET) $(OBJS) $(DEPS)

-include $(DEPS)
```

That's already it, after typing

```bash
make
```

your agent should build and be ready to use.

> **NOTE**
>
> If you copied the code directly from the browser make sure your IDE
> inserts tabs and not spaces (as Makefiles need tabs).

Try it by simply running the executable in an all-default setting (using the
vrpc.io broker and the free `public.vrpc` domain):

```bash
./vrpc-foo-agent
```

Once you see the line

```bash
Connecting to the MQTT server... [OK]
```

appearing in your terminal, you made it and your C++ code is remotely callable!

Convince yourself and point your browser to
[live.vrpc.io](https://live.vrpc.io). Log in using `public.vrpc` as domain name
and leave the token empty. You should see your agent online (it uses your user-,
host- and platform name).

Or call your code from another piece of code written in e.g. Javascript. Just
follow the `Node.js Client` example.

> **NOTE**
>
> As you are using the free but public `public.vrpc` domain your code
> may be executed by anyone that uses your agent name.
> While convenient for quick testing or examples like this, it's obviously
> not an option for production settings. Please refer to the optional steps A-C
> if you want to make the communication between your agents and clients private.

## Optional steps to make your communication private

### STEP A: Create a free VRPC account

If you already have an account, simply skip this step.

If not, quickly create a new one by clicking on "CREATE NEW ACCOUNT" using the
[VRPC App](https://app.vrpc.io). It takes less than a minute and the only thing
required is your name and a valid email address.

### STEP B: Create a free domain

If you already have a domain, simply skip this step.

If not, navigate to the `Domains` tab in your VRPC app and click *ADD DOMAIN*,
choose a free domain and hit *Start 30 days trial* button.

### STEP C: Test VRPC installation and connectivity

For any agent to work, you must provide it with a valid domain and access
token. You get an access token from your VRPC app using the `Access Control` tab.

Simply copy the *defaultAgentToken* or create a new one and use this.

Having that you are ready to make the communication to your agent private:

```bash
./vrpc-test-agent -d <yourDomain> -t <yourToken>
```

In case of success you should see an output similar to this:

```bash
Domain          : <yourDomain>
Agent ID        : <yourAgent>
Broker URL      : ssl://vrpc.io:8883
------------------
Persistance     :
Clean Session   : 1
Connect Timeout : 10
Keep Alive      : 120
Server Auth     : 0
Connecting to the MQTT server... [OK]
```

Now, your agent code runs under your private domain and anyone wanting to
communicate to it needs to be in the same domain and having a valid access
token to do so. Using the [VRPC App](https://app.vrpc.io) you may even generate access tokens
with detailed access rights down to a per-function level.
