# Building GnoSocial: A Decentralized Microblogging Platform on Gno.land

## Introduction

[Gno.land](https://gno.land/) is an open source smart contract system, built using Gno, an interpreted, fully deterministic language based on [and extremely similar to](https://docs.gno.land/reference/go-gno-compatibility) Go. It empowers developers to build sophisticated smart contracts using the simple and familiar syntax of Go, a language used daily by millions of developers.

In this tutorial, we are creating GnoSocial, a decentralized microblogging platform inspired by X (formerly Twitter), using Gno.land. We'll explore how blockchain technology can revolutionize social media by ensuring transparency, user data ownership, and censorship resistance.

## Prerequisites

* Basic understanding of programming concepts
    
* Install a wallet compatible with Gno.land -- currently this is the [Adena Wallet](https://www.adena.app/) -- and create an address if you do not already have one
    
* Familiarity with [Go](https://go.dev/doc/tutorial/getting-started) syntax (as Gno is similar to Go)
    
* Access to [Gno Playground](https://play.gno.land/) and [Gno Studio Connect](https://gno.studio/connect)
    

With these prerequisites in hand, we are ready to build the core of our decentralized social media platform.

## Step 1: Setting up the Project in Gno Playground

The Gno Playground provides a browser based editor and sandbox for Gno code, allowing a developer to write code, run tests, share their code, and deploy it, all within a single web based tool.

To get started, navigate to the [Gno Playground](https://play.gno.land/).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720172979620/34d98216-0451-4150-8f47-7c6f8ee22fa1.png align="center")

The Playground will create an initial file for you. Add a new file named `gnosocial.gno` and delete the `package.gno` file.

In your new `gnosocial.gno` file, start with the following code:

```go
package gnosocial

import (
    "gno.land/p/demo/ufmt"
    "std"
    "strings"
    "time"
)

type Post struct {
    Author  std.Address
    Content string
    Time    time.Time
}

type DirectMessage struct {
    Sender   std.Address
    Receiver std.Address
    Content  string
    Time     time.Time
}

var posts []Post
var directMessages []DirectMessage

// ... (rest of the code)
```

For the core functionality of Gnosocial, we just need to track posts and direct messages. Gno, just like Go, allows the definition of data structures.

For posts, our platform will track authorship via one's wallet address, the content of the post, and the time that the post was created. Direct messages are similar, except that both sender and receiver are tracked.

## Step 2: Implementing the Post Content Function

The core of any social media platform is its content. Let's create a function to handle posts.

```go
// Post your thoughts.
//
// Enter a post below, and then press the `Call` button.
//
// The post will be recorded along with your address for everyone else to read.
func CreatePost(content string) {
    newPost := Post{
        Author:  std.GetOrigCaller(),
        Content: content,
        Time:    time.Now(),
    }
    posts = append(posts, newPost)
}
```

This function takes the provided content, and uses it to create a new record, owned by the caller of the function, with that content, the address of the owner, and time time that it was created.

## Step 3: Implementing the Direct Message Function

Another essential feature of any social platform is private messaging. Let's write a function to enable the sharing of messages between a sender and a receiver.

```go
// Talk with someone else! Enter their address, and the message that you want
// to send to them below, and then press the `Call` button.
func SendDirectMessage(receiver std.Address, content string) {
    newDM := DirectMessage{
        Sender:   std.GetOrigCaller(),
        Receiver: receiver,
        Content:  content,
        Time:     time.Now(),
    }
    directMessages = append(directMessages, newDM)
}
```

This function is very similar to `createPost`, differing only in that it also accepts, and stores, a receiver for the content.

## Step 4: Implementing the Render Functions

In a Gno.land realm, the `Render` function can be used to request a markdown rendering of the realm without requiring a transaction. For Gnosocial, the `Render` function will display either all posts, or all DMs, based on the argument that is passed into it.

To simplify code understanding and maintenance, we'll create two separate functions for rendering posts and direct messages. The first of these is `RenderPosts`.

```go
// Call this to see all of the current posts.
func RenderPosts() string {
	var output strings.Builder
	output.WriteString("# All Posts")
	for _, post := range posts {
		output.WriteString(ufmt.Sprintf("* %s @ %s: %s", post.Time.String(), post.Author, post.Content))
	}
	return output.String()
}
```

This function iterates through the posts, creating an output string, in Markdown, of all posts, the address of the author, and the time of authorship.

Next, let's write `RenderDMs`.

```go
// Call this to see your messages.
func RenderDMs() string {
	var output strings.Builder
	caller := std.GetOrigCaller()

	if caller == "" {
		output.WriteString("* All Direct Messages")
	} else {
		output.WriteString("* Your Direct Messages")
	}

	for _, dm := range directMessages {
		if caller == "" || dm.Sender == caller || dm.Receiver == caller {
			output.WriteString(ufmt.Sprintf("* From: %s to: %s @ %s: %s", dm.Sender, dm.Receiver, dm.Time.String(), dm.Content))
		}
	}
	return output.String()
}
```

This function was implemented with a little more functionality. If it is invoked in a context where `GetOrigCaller` returns a caller, it will return only the direct messages which were sent to, or from, that caller. When `GetOrigCaller` doesn't return a caller, such as when called through `Render`, as illustrated below, all of the Direct Messages will be returned.

## Step 5: Implementing the Main Render Function

Finally, let's implement the main Render function. This function will do one of two things, depending on the path that it is given when it is called.

```go
// Calling Render() with `/posts` will return an HTML string with all of the current posts.
// Calling Render() with `/dms` will return an HTML string with all of the current direct messages.
func Render(path string) string {
	command := ""
	if idx := strings.Index(path, "/"); idx >= 0 {
		command = path[idx+1:]
	}

	switch command {
	case "posts":
		return RenderPosts()
	case "dms":
		return RenderDMs()
	default:
		return "Welcome to GnoSocial! Use /posts to view all posts or /dms to view your messages."
	}
}
```

If `Render` is called with a path ending in `/posts`, it will call the `RenderPosts` function, returning all posts. If it is called with a path ending in `/dms`, it will call the `RenderDMs` function, returning all DMs. If called with anything else, it returns a welcome sentence with some simple instructions.

## Step 6: Deploying It

Now, let's see it in action! From the Gno Playground, you can deploy your code to any network connected to your wallet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720182875730/aacd7bb4-e90b-4c83-a423-7534a703a749.png align="center")

Connect your wallet to Testnet 3. If you Gnot balance (the Gno.land token) is low, consider topping it up using the [Gno.land faucet](https://gno.land/faucet) before proceeding.

To Deploy, click on the `Deploy` link in the top menu bar:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720183067063/9f704631-a126-455a-a0b0-ae24c927cde0.png align="center")

You should now see something similar to the image below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720183138035/edccc37a-13dd-4952-86ed-7aef1b86ac7c.png align="center")

The Path reflects where your realm will be deployed. Provide it with a username for yourself, which serves as a namespace for your realms. The name of the realm will automatically be set to match the package name.  
Then, press `Deploy`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720183255537/7d6ac63a-78e2-4562-a128-342b1de21d75.png align="center")

Your wallet will ask you to confirm the transaction:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720183372310/e090d305-6f6f-4001-9e50-4fb0d955515b.png align="center")

On success, you will be rewarded by the deployment drop down displaying "Deployment Successful" and providing you with a link to interact with your newly deployed realm within the Gno Studio. If there was a problem with your code, you will see a stack trace with the error message. Go back and double check that you have copied all of the code in this tutorial faithfully, and then try again.

## Step 7: Interact with My Deployed Version

Now comes the fun part -- seeing it work!

If you click on the link provided after the successful deployment, you will be taken to the Gno Studio, where you will be connected with your newly deployed realm. If you are just reading along, you can go to my version here:

[https://gno.studio/connect/view/gno.land/r/kirk\_haines/gnosocial?network=test3](https://gno.studio/connect/view/gno.land/r/kirk_haines/gnosocial?network=test3)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720184756661/7edd4918-0892-43d1-97bb-5c1ecb7a33a4.png align="center")

Gno Studio Connect lets you interact with a deployed realm via your web browser. Scroll down until you see something similar to the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720184967154/b08e2c04-7032-4868-a4dc-cda71f66c784.png align="center")

These are the five callable functions that are defined in our Gnosocial realm. Go ahead and use one to create a post. Click on `CreatePost`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720185140210/59d91e72-60c4-4d4a-bed6-167c43220ebd.png align="center")

Type your post, and when you are finished, press the `Call` button. Once again, your wallet will pop up a dialog asking for you to confirm the transaction. This is because any action which may change blockchain state much be in the context of a transaction, and thus must be paid for with Gnot.

Congratulations, you have made your first Gnosocial post!

Now, do the same with the `SendDirectMessage` function. Send me a direct message. My address is:

g1whzkakk4hzjkvy60d5pwfk484xu67ar2cl62h2

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720186148086/74208f63-3ce9-4b01-94f0-30f9927d7118.png align="center")

Now, let's take a look at the posts and direct messages. Remember that we created those helper functions, `RenderPosts` and `RenderDMs`. These functions, if called directly, cost Gnot to call, because it is possible that they could change chain state. Go ahead and try it.

When you call `RenderDMs`, pay attention to the string that is returned. It should start, `* Your Direct Messages`. When called directly, within the context of a transaction, information about the caller is available, so the code can return information that is relevant only to the caller.  
  
Now invoke the `Render` function. Notice that although there is a field to provide an argument, the button to invoke the function says "Eval" instead of "Call". This Gnosocial `Render` function will return all posts, or all dms, depending on how it is called. So, enter `/dms` into the field, and press `Eval`. You will get back something akin to this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720187455710/1bca4e8a-2609-4db2-abf3-33401c5cf158.png align="center")

See how it starts differently, with `All Direct Messages`? If you refer back to the code above, you will see that the choice of whether this happens or not is determined by whether a caller can be determined. When Render is invoked, it runs outside of a transaction, so no caller data is present from `GetOrigCaller()`.

## Engagement: The Future of Social Media

GnoSocial is a very trivial implementation of the basic functions that make up a platform like X/Twitter. Nonetheless, this does serve to illustrate how simple the code is to implement and understand, when using Gno, and it is a kernel that could be expanded into a real platform, given a little time.

Beyond that, there are deeper reasons why a social media platform implemented in a smart contract system like [Gno.land](http://Gno.land) could offer significant advantages for users.

### Data ownership:

In this trivial example, the Gnosocial realm owns all of the data. However, a logical evolution of this platform would be to create a Gnosocial User Interface realm that individual users would deploy. If this user interface realm was built to contain all of one's data, then each individual who interacts with Gnosocial would effectively control their data and the access to it.

### Censorship resistance

No central authority can arbitrarily remove content. No entity could compel Gnosocial to delete content (though controls could be put into place on the dApp side of things to limit content visibility in the application itself).

### Privacy

While it is true that blockchains, by their nature, tend to expose data, through proper use of encryption within the Gnosocial User Interface, or maybe even by leveraging adjunct technologies like ZK Proofs, the transparency of blockchain could effectively be balanced against the need to keep private data private, and the ability to limit the data that Gnosocial itself is privy to.

As you build and interact with GnoSocial, consider how these features could reshape online interactions and empower users in ways traditional social media platforms find difficult.

## Conclusion

In this tutorial, we've built GnoSocial, a decentralized microblogging platform using [Gno.land](http://Gno.land). We've implemented key features like posting content, sending direct messages, and rendering different views. By using Gno Studio Connect, users can interact with GnoSocial in a decentralized manner, showcasing the potential of blockchain technology in reshaping social media.

This tutorial provides a foundation for building decentralized social applications. Readers are encouraged to expand on this basic structure by adding features like commenting, liking posts, or implementing a follower system.
