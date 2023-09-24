# Google Summer of Code 2023, RocketChat

<div>
    <a href="https://summerofcode.withgoogle.com/projects/#6521788818784256"><img src="https://i.imgur.com/pgkUceb.png" width="650" alt="google-summer-of-code"></a>
    <br>
    <b> 
        <p>
        Create a ready-to-go easy to embed mini-chat React component.
        </p>
    </b>
</div>

I worked on a project called [EmbeddedChat](https://github.com/RocketChat/EmbeddedChat) which is an in-app chat solution that utilizes the RocketChat chat engine through its REST and real-time APIs to support powerful chat features like reactions, online presence, typing status, threads, and much more.

I would maintain this repository as the final report summary of my GSoC 2023 project and a quick guide for all future GSoC aspirants.

## ⭐ Project Abstract
The goal of the project is to make a ready-to-use chat solution that could be integrated into any website, web app, or app. This project was a major refactoring and enhancement for the EmbeddedChat 2022 project.

## 🚢 Deliverables
- Improve authentication - support all OAuth services
- Move to a mono repo - auth, api, react, react-native, HTML embed
- HTML embed feature
- Theming
- Improving API
- Support for slash commands
- Migrating from the fuselage to our own minimal components

## Demo

### Sneak Peak
EmbeddedChat integrated into my esportsweb.in website.
![Screenshot from 2023-09-24 17-52-05](https://github.com/abhinavkrin/GSoC-RocketChat-2023/assets/15830206/da7e7e9f-f944-4c9d-8b1d-c882d081c4ab)

### Moving to mono repo
EmbeddedChat's new mono repo structure
![structure](https://github.com/abhinavkrin/GSoC-RocketChat-2023/assets/15830206/51b8ef1f-13c6-47f3-bac7-e48dd3b16197)
- *auth* - The auth package includes functions to easily log into a Rocket chat server. Though it is used by embeddedchat's react and react-native client, developers can use this package for their own use cases.
- *api* - The api package includes functions that are all required to create a chat application using the Rocketchat server. It has functions like connect, login, sendMesage, pinMessage, starMessage, deleteMessage, triggerBlockAction, etc. to perform various operations. One can listen to new/updated message events by attaching event listeners using `addMessageListener`. There are other event listeners which could be added using `addMessageDeleteListener`, `addTypingStatusListener`, `addActionTriggeredListener`, `addUiInteractionListener`.
- *react* - The react package includes the react components to integrate EmbeddedChat.
- *react-native* - The react-native project aims at using EmbeddedChat in react native mobile apps.
- *htmembed* - With this project EmbeddedChat could be integrated into any web app by simply embedding an HTML snippet.

### HTML Embedd Feature
Simple integrate embedded chat by pasting html snippet into your website
```
<div id="embeddedchat"></div>
      <script src="http://127.0.0.1:4001/embeddedchat.js"></script>
      <script>
        // all props for the EmbeddedChat of @embeddedchat/react will go here
		// The config will be directly applied as props for the EmbeddedChat Component
        const config = {
            host: 'http://localhost:3000',
    		roomId: 'GENERAL',
            isClosable: true,
            setClosableState: true,
            moreOpts: true,
            channelName: 'general',
            anonymousMode: true,
            headerColor: 'white',
            toastBarPosition: 'bottom-end',
            showRoles: true,
            showAvatar: false,
            enableThreads: true,
            auth: {
                flow: 'MANAGED',
            },
        }
        EmbeddedChat.renderInElementWithId(config, 'embeddedchat')
      </script>
```
HTML Embedded in action
![htmlembed](https://github.com/abhinavkrin/GSoC-RocketChat-2023/assets/15830206/1db54c74-095c-42b6-8cc4-e0e320af8748)

### Theming
We can customize EmbeddedChat by passing a custom theme object. Hence, it could take the look and feel of the app or website. We can also customize components by custom stylesheet or passing custom class names through the theme object.
![Customizing using theme](https://github.com/abhinavkrin/GSoC-RocketChat-2023/assets/15830206/8dd8f797-0ede-4f87-996d-42d23ddcbba0)

### Improving API
Our `api` package exposes the `EmbeddedChatApi` class. It comes with a bunch of APIs that could be used to login, send, pin, edit, star or delete message, attach listeners for realtime events. It has the following structure:
```
class EmbeddedChatApi {
    host: string;
    rid: string;
    rcClient: Rocketchat;
    onMessageCallbacks: ((message: any) => void)[];
    onMessageDeleteCallbacks: ((messageId: string) => void)[];
    onTypingStatusCallbacks: ((users: string[]) => void)[];
    onActionTriggeredCallbacks: ((data: any) => void)[];
    onUiInteractionCallbacks: ((data: any) => void)[];
    typingUsers: string[];
    auth: RocketChatAuth;
    constructor(host: string, rid: string, { getToken, saveToken, deleteToken, autoLogin }: IRocketChatAuthOptions);
    setAuth(auth: RocketChatAuth): void;
    getAuth(): RocketChatAuth;
    getHost(): string;
    googleSSOLogin(signIn: Function, acsCode: string): Promise<any>;
    login(userOrEmail: string, password: string, code: string): Promise<{
        status: string;
        me: any;
    } | undefined>;
    logout(): Promise<void>;
    connect(): Promise<void>;
    addMessageListener(callback: (message: any) => void): Promise<void>;
    removeMessageListener(callback: (message: any) => void): Promise<void>;
    addMessageDeleteListener(callback: (messageId: string) => void): Promise<void>;
    removeMessageDeleteListener(callback: (messageId: string) => void): Promise<void>;
    addTypingStatusListener(callback: (users: string[]) => void): Promise<void>;
    removeTypingStatusListener(callback: (users: string[]) => void): Promise<void>;
    addActionTriggeredListener(callback: (data: any) => void): Promise<void>;
    removeActionTriggeredListener(callback: (data: any) => void): Promise<void>;
    addUiInteractionListener(callback: (data: any) => void): Promise<void>;
    removeUiInteractionListener(callback: (data: any) => void): Promise<void>;
    handleTypingEvent({ typingUser, isTyping }: {
        typingUser: string;
        isTyping: boolean;
    }): void;
    updateUserNameThroughSuggestion(userid: string): Promise<any>;
    updateUserUsername(userid: string, username: string): Promise<any>;
    channelInfo(): Promise<any>;
    close(): Promise<void>;
    getMessages(anonymousMode?: boolean, options?: {
        query?: object | undefined;
        field?: object | undefined;
    }): Promise<any>;
    getThreadMessages(tmid: string): Promise<any>;
    getChannelRoles(): Promise<any>;
    sendTypingStatus(username: string, typing: boolean): Promise<void>;
    sendMessage(message: any, threadId: string): Promise<any>;
    deleteMessage(msgId: string): Promise<any>;
    updateMessage(msgId: string, text: string): Promise<any>;
    starMessage(mid: string): Promise<any>;
    unstarMessage(mid: string): Promise<any>;
    getStarredMessages(): Promise<any>;
    getPinnedMessages(): Promise<any>;
    pinMessage(mid: string): Promise<any>;
    unpinMessage(mid: string): Promise<any>;
    reactToMessage(emoji: string, messageId: string, shouldReact: string): Promise<any>;
    reportMessage(messageId: string, description: string): Promise<any>;
    findOrCreateInvite(): Promise<any>;
    sendAttachment(file: File, fileName: string, fileDescription?: string, threadId?: undefined): Promise<any>;
    me(): Promise<any>;
    getChannelMembers(): Promise<any>;
    getSearchMessages(text: string): Promise<any>;
    triggerBlockAction({ type, actionId, appId, rid, mid, viewId, container, ...rest }: any): Promise<any>;
    getCommandsList(): Promise<any>;
    execCommand({ command, params }: {
        command: string;
        params: string;
    }): Promise<any>;
}
```
Demo playgroud for api package
![playground_api](https://github.com/abhinavkrin/GSoC-RocketChat-2023/assets/15830206/44eea372-1acc-464d-9e2a-a38ae5a0a030)


