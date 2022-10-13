# ToolkitMessenger



Improved messenger APIs 📬
Another commonly used feature in the MVVM Toolkit is the IMessenger interface, which is a contract for types that can be used to exchange messages between different objects.

This can be useful to decouple different modules of an application without having to keep strong references to types being referenced. It’s also possible to send messages to specific channels, uniquely identified by a token, and to have different messengers in different sections of an application.

The MVVM Toolkit provides two implementations of this interface:

WeakReferenceMessenger: which doesn’t root recipients and allows them to be collected. This is implemented through dependent handles, which are a special type of GC references that allow this messenger to make sure to always allow registered recipients to be collected even if a registered handler references them back, but no other outstanding strong references to them exist.
StrongReferenceMessenger: which is a messenger implementation rooting registered recipients to ensure they remain alive even if the messenger is the only object referencing them.
Here’s a small example of how this interface can be used:

// Declare a message
public sealed record LoggedInUserChangedMessage(User user);

// Register a recipient explicitly...
messenger.Register<MyViewModel, LoggedInUserChangedMessage>(this, static (r, m) =>
{
    // Handle the message here, with r being the recipient and m being the
    // input message. Using the recipient passed as input makes it so that
    // the lambda expression doesn't capture "this", improving performance.
});

// ...or have the viewmodel implement IRecipient<TMessage>...
class MyViewModel : IRecipient<LoggedInUserChangedMessage>
{
    public void Receive(LoggedInUserChangedMessage message)
    {
        // Handle the message here
    }
}

// ...and then register through the interface (other APIs are available too)
messenger.Register<LoggedInuserChangedMessage>(this);

// Send a message from some other module
messenger.Send(new LoggedInUserChangedMessage(user));
The messenger implementations in this new version of the MVVM Toolkit have been highly optimized in .NET 6 thanks to the newly available public DependentHandle API, which allows the messenger types to both become even faster than before, and also offer completely zero-alloc message broadcast. Here’s some benchmarks showing how the messengers in the MVVM Toolkit fare against several other equivalent types from other widely used MVVM libraries:
