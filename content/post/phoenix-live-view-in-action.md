+++
title = "Phoenix LiveView in Action"
description = "Building Interactive Web Application with Elixir's Phoenix Framework"
tags = [
    "elixir", "phoenix", "webdev"
]
date = 2023-12-08T10:55:00Z
author = "Scorpil"
images = [
  "/img/phoenix-live-view-in-action/phoenix-liveview.png",
  "/img/phoenix-live-view-in-action/liveview-genserver-diagram.png",
]
+++

_TL;DR: I used Phoenix LiveView to develop a <a href="https://estim8.pro" target="_blank">simple planning poker app</a> and found it exceptionally well-suited for the task._

The Elixir programming language, and arguably its strongest showcase, the Phoenix Framework, are technologies that, in my mind, are constantly on the edge of mainstream popularity. Both the language and the framework are exotic enough to present a significant learning challenge, yet many of their innovative features, such as built-in fault tolerance, cluster-awarness and real-time communication capabilities, justify the effort.

I've been following the developments in the Phoenix Framework for the last five years, deeply intrigued by its evolution (I even went so far as to [write a blog post](https://scorpil.com/post/things-elixirs-phoenix-framework-does-right/) about its strengths a few years ago). While I haven't yet had the chance to work with Phoenix in a production setting on a large scale, I regularly experiment with it hands-on in various pet projects.

A few months ago, my team and I found ourselves in need of a straightforward scrum-poker tool. Although there are several such services available online, many of them come with drawbacks like mandatory registration, paid accounts, and an overload of non-essential features that detract from the main functionality. So, I took this situation as the perfect opportunity to try out one of Phoenix's most interesting features: LiveView.

![stylized Phoenix Framework logo](/img/phoenix-live-view-in-action/phoenix-liveview.png)

## What is Phoenix Live

The most common web application architecture today typically involves server-client communication via a REST API, with the final webpage assembled by client-side JavaScript. This approach offers notable advantages, chief among them being a clear separation of concerns. However, it's not without its drawbacks. Dynamically generated web pages can pose challenges for search engine indexing, and caching the final webpage layout becomes impractical. Additionally, managing updates on the client side can be complex and resource-intensive. A significant concern is that larger client-side applications often replicate server-side state, increasing complexity and the likelihood of synchronization errors.

Server-Side Rendering (SSR) addresses these limitations by rendering webpages on the server and sending a complete HTML payload to the client. This method is efficient for static content, but integrating client-side interactivity can be challenging. Frameworks like Next.js aim to bridge this gap by offering a hybrid development environment that integrates front-end and back-end components seamlessly. However, these frameworks are still fundamentally built around the idea that the request-response cycle is the primary mode of interaction between the user and the web server. While this pattern covers many use cases, modern web applications often need to update information on the page without direct user intervention. Scenarios such as live sports scores, stock market tickers, or real-time notifications are examples where information must be dynamically updated.

There are a number of techniques designed for this less frequent, yet crucial, information delivery pattern, the most common of which are WebSockets. WebSockets provide a two-way, stateful communication protocol for web applications. While it's possible to integrate WebSocket communications with before-mentioned Next.js, the framework does not facilitate such interaction in any way: developer is responsible for maintaining the connection, developing communication protocol and updating the communication state.

Phoenix LiveView is a feature that can be viewed as an attempt to merge Server-Side Rendering with a client-side interactivity in a seamless way, avoiding boilerplate code and state duplication. It accomplishes this by not only delivering the initial page as a statically rendered payload but also by establishing a persistent WebSocket connection to facilitate server-side-rendered application updates.

This simple idea makes for a compelling set of properties:
- Initial page load is a static HTML, so it can be easily indexed, cached and distributed through CDN.
- The state is maintained on the server, resulting in a lightweight client side that only manages the WebSocket connection and *content-agnostic* update propagation.
- The framework handles the underlying transport and content reconciliation, freeing the developer to focus on application-specific logic.
- As the server and client share the same state, all updates are automatically reflected on the client side. This is handled uniformly, whether the changes are user-initiated or occur externally (unlike in classical architectures where user-initiated changes typically go through an API, while external changes are pushed through a WebSocket).

It's worth noting that Phoenix offers a synergistic set of features for real-time communication, which LiveView leverages effectively.

## From Concept to Practice: LiveView in the Project

The guiding principle of my homemade scrum-poker project was simplicity in both user experience and developer effort. For users, I aimed to create a tool that required minimal effort to engage with, ensuring a straightforward and efficient experience. Simplicity from a development standpoint was equally crucial. With limited time available, I needed a solution that provided a stable and efficient platform, supporting essential functionalities without extra complexity.

The feature set was distilled to the following essentials:
- Users can create a shared room for their team without the need for registration or any additional steps.
- Joining an existing room is equally hassle-free, requiring no registration.
- Rooms are designed to be ephemeral: existing only when at least one user is present and not maintained in any permanent state.
- All users hold equal status in a room, eliminating the need for administrative roles. This simplifies the implementation and suits the intended audience of collaborative teams.
- Users have the option to change their display name for identification purposes in the room, but it's not mandatory, allowing for complete anonymity.
- The option to mark oneself as an observer is available, indicating a non-voting role.
- A clear display of users' statuses (voting, not voting, observing) is available, along with their display names.
- Room descriptions can be set for context, like identifying the ticket being estimated.
- A selection of card decks is available for estimations, including typical scrum-style decks, real Fibonacci sequences, T-shirt sizes, and positive integers.
- The estimation process is two-phased: initially, votes are cast privately, followed by a simultaneous reveal of all votes, including mean and median values.

Given these features, the primary challenge was ensuring real-time synchronization of the room state among all participants, a crucial aspect of the service. My goal was to minimize the delay between a user's action and its reflection on their colleagues' screens as much as possible.

Phoenix LiveView proved to be the ideal choice for this task, and I eagerly embraced the opportunity to deepen my hands-on experience with this feature. It’s important to note, though, that while I’ve utilized Phoenix in a few small projects, I don’t consider myself an expert. Therefore, some of the design decisions I made might not be optimal, reflecting my learning curve with the technology.

## Inside LiveView: Building Blocks of Interactive Apps

Web interfaces in Phoenix are constructed using Components, which are just functions receive attributes (or assigns, as referred to in Phoenix) and return a struct containing rendered HTML and some metadata. This approach is similar to how reusable Components work in front-end frameworks like ReactJS and VueJS. Phoenix component might look

For standard, non-live-enabled web pages, Phoenix employs **Views**. View is a module that is responsible for all tasks necessary to build a complete page: retrieving data from external sources, assembling layout templates, combining Components and so on. They play a crucial role in the application's architecture, acting as the bridge between the data and its presentation on the client side.

For interactive applications, Phoenix introduces **LiveView** and **LiveComponent**. LiveView is a specialized Phoenix View that enables real-time updates via WebSockets. It offers lifecycle hooks and event handlers for various stages of a page's lifecycle, including during initial rendering, WebSocket connections and disconnections, custom user-initiated events, and events from other parts of the system.

Similarly, LiveComponent is a type of Component that supports lifecycle hooks and can handle events. It is particularly useful for managing complex states in larger applications. However, for my smaller project, I found them less necessary. In my setup, LiveView manages the entire state, while simple stateless Components are used for rendering.

After considering the app's structure, I opted for just two pages: a static landing page with a logo, memorable motto, and a 'Create a Room' button, built using a standard View, and an interactive 'room' page, developed with LiveView.

Sharing state between different user connections in Phoenix doesn't happen automatically, but the framework provides all the necessary building blocks to create a custom solution. My state management system relied on two key Phoenix techniques:
- GenServer for creating a room registry. This abstraction over Elixir/Erlang processes is beneficial in Phoenix due to its automatic supervision, ensuring processes are monitored and relaunched as needed. My RoomRegistry GenServer is tasked with creating new rooms, maintaining a list of active rooms, and cleaning up inactive ones.
- PubSub for system-wide communication, including backend-to-frontend through integrated WebSockets. PubSub is crucial for horizontal scalability in stateful Phoenix applications, enabling cluster-based Phoenix nodes to distribute load and state.

When a new user visits a room URL, LiveView interacts with RoomRegistry to either create a new room or join an existing one. RoomRegistry keeps track of connected users and propagates any state changes through LiveView to RoomRegistry, which then broadcasts the updated state to all room participants simultaneously. User disconnects are also reported to RoomRegistry, allowing it to remove room metadata and free up memory when the user count drops to zero.

![architecture diagram](/img/phoenix-live-view-in-action/liveview-genserver-diagram.png)

To better understand this flow, lets examine a single simple feature: setting the username. The username field is defined at [/lib/estim8_web/live/room_live.html.heex](https://github.com/Scorpil/Estim8/blob/4c33ff35db80b66196b443941f69e2d77ac5d37d/lib/estim8_web/live/room_live.html.heex#L19) file. Phoenix uses `phx-` HTML pseudo-attribute to bind form events to corresponding controller handlers:

```HTML
  <form id="name" phx-change="namechange" phx-submit="namechange" class="flex flex-row items-center">
    <label for="nameinput" class="text-gray-500 text-sm pr-1 hidden md:block">Your name: </label>
    <input id="nameinput" name="name" class="h-8" type="text" placeholder="Name" value={@me.name} phx-debounce="2000"/>
  </form>
```

Thanks to the `phx-change` and `phx-submit` attributes in the form, the LiveView controller is notified of the `namechange` event with every modification in the input field and upon form submission (e.g., when Enter is pressed). Additionally, the `phx-debounce` attribute is used to rate-limit these events, which helps in reducing the number of unnecessary requests. `@me` in this snippet is a map containing attributes of the current user.

This LiveView component handler is triggered in response to the `namechange` event ([/lib/estim8_web/live/room_live.ex](https://github.com/Scorpil/Estim8/blob/4c33ff35db80b66196b443941f69e2d77ac5d37d/lib/estim8_web/live/room_live.ex#L71)):

```elixir
  def handle_event("namechange", %{"name" => name}, socket) do
    Estim8.Room.user_namechange(socket.assigns.room, socket.assigns.me.id, name)
    socket = assign(socket, %{
      me: Estim8.User.update_name(socket.assigns.me, name)
    })
    {:noreply, push_event(socket, "namechange", %{name: name})}
  end
```


`Estim8.Room` is a just a wrapper for dispatching actions to the GenServer. It tells RoomRegistry to update users' name in the list of users for the current room. Once done, `@me` parameter gets updated with the new name (this value is local to the user, so it's not managed by the GenServer), and `namechange` event gets sent back to the fronted (more on this later).

`user_namechange` function, asks GenServer to update the state of the room, then `broadcast` function uses PubSub to tell LiveView about the new room state ([/lib/estim8/room.ex](https://github.com/Scorpil/Estim8/blob/4c33ff35db80b66196b443941f69e2d77ac5d37d/lib/estim8/room.ex#L107)):

```elixir
  def user_namechange(room, user_id, new_name) do
    Agent.update(room, fn (state) ->
      state
      |> Map.update!(:users, fn (users) -> Map.update!(users, user_id, fn (user) -> Estim8.User.update_name(user, new_name) end) end)
    end)
    broadcast(room)
  end
```

The last piece of the puzzle is LiveView receiving the room state and updating UI for every room participant ([/lib/estim8_web/live/room_live.ex](https://github.com/Scorpil/Estim8/blob/4c33ff35db80b66196b443941f69e2d77ac5d37d/lib/estim8_web/live/room_live.ex#L87)):

```elixir
  def handle_info({:update, state}, socket) do
    {:noreply, assign(socket, Map.merge(
      state,
      %{
        me: state.users[socket.assigns.me.id],
        deck: Map.get(@deck_list, state.settings.deck_id, Estim8.Deck.empty()),
        settings_form: to_form(Estim8.RoomSettings.changeset(%Estim8.RoomSettings{}, state.settings)),
      }
    ))}
  end
```

To enhance the user experience, I wanted the user's chosen name to persist between sessions. Phoenix doesn't have any special way of working with Local Storage, but the autogenerated file `/assets/js/app.js`, which establishes the WebSocket connection, is perfect for this kinds of client-side customizations directly through local JavaScript.

Since the namechange event is [forwarded to the fronted in the LiveView](https://github.com/Scorpil/Estim8/blob/4c33ff35db80b66196b443941f69e2d77ac5d37d/lib/estim8_web/live/room_live.ex#L76), we can attach a simple callback to store the name into local storage on every change:
```js
window.addEventListener("phx:namechange", (e) => {
  localStorage.setItem("userName", e.detail.name)
})
```

Similarly, I fetch the name from the local storage before connecting to the WebSocket. The name, together with a randomly generated userId are passed to the LiveView during the initial connection phase.
```js
let userId = localStorage.getItem("userId")
if (userId === null) {
  userId = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15)
  localStorage.setItem("userId", userId)
}

let userName = localStorage.getItem("userName")
if (userName === null) {
  userName = "Anonymous"
}
```

We can subscirbe to the `phx:namechange` events in JavaScript 

## Unpacking Lessons Learned

With this blog post, and with my work on Estim8 project I only scratched the surface of whats possible with Phoenix LiveView. This feature stands out for its powerful functionality and elegant design. However, its learning curve can be steep. When starting a project with Phoenix, it’s common to quickly become reliant on several features unique to Elixir and Phoenix, such as GenServers and PubSub. These features perform the heavy lifting, and once you overcome the initial learning challenges, they enable the creation of reliable, low-latency interactive web applications in a remarkably short time.

Most importantly, I enjoyed the process of diving into this new paradigm of web application architecture. I think we will see more experiments like this in the future in other technologies as well, making interactive apps simpler and less boilerplate-heavy.

I encourage you to take Phoenix LiveView for a spin when you have a fitting project in the works. This unique tool is worth keeping in your toolset.

Please also check out the result of my work: <a href="https://estim8.pro/" target="_blank">Estim8</a>. It's source code is available [on GitHub](https://github.com/Scorpil/Estim8/).