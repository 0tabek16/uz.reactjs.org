---
id: hooks-custom
title: Oʻzingizni hukingizni yaratish
permalink: docs/hooks-custom.html
next: hooks-reference.html
prev: hooks-rules.html
---

*Huklar* - React 16.8'ga kiritilgan yangilik. Ular sizga klass yozmasdan turib holat (state) va React'ning boshqa qulayliklarini ishlatishga yordam beradi.

Oʻzingizni hukingizni yaratish orqali komponentning mantiqiy qismini qayta ishlatsa boʻladigan funksiyaga ajrata olasiz.

[Taʼsir hukini ishlatish](/docs/hooks-effect.html#example-using-hooks-1)ni oʻrganish davomida, chat dasturida doʻstimizni onlayn yoki oflayn ekanligini koʻrsatadigan komponentini koʻrgan edik:

```js{4-15}
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Yuklanyapti...';
  }
  return isOnline ? 'Onlayn' : 'Oflayn';
}
```

Qani endi chatimizda odamlarning roʻyxati ham bor deylik va biz onlayn boʻlgan foydalanuvchilarni ismini yashil rangda koʻrsatmoqchimiz. Bunda, shunchaki tepadagiga oʻxshagan mantiqiy qismni `FriendListItem`ga koʻchirib qoʻya olamiz, lekin bu bekam-ko'st boʻlmaydi:

```js{4-15}
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

Oʻrniga, ushbu mantiqiy qismni `FriendStatus` va `FriendListItem` oʻrtasida taqsimlasak boʻladi.

Azaldan, Reactʼda holat qatnashgan mantiqiy qismlarni ikki mashhur yoʻl bilan taqsimlay olasiz: [chizuv kiritmalari](/docs/render-props.html) (render props) yoki [yuqori darajali komponent](/docs/higher-order-components.html) (higher-order components). Huklar bilan esa, bu muammoni (DOM) daraxtga ortiqcha komponent qoʻshmasdan hal qilsa boʻladi.

## Qoʻlbola hukni ajratish {#extracting-a-custom-hook}

JavaScriptʼda mantiqiy qismni ikki funksiya bilan ulashish uchun, uni uchinchi funksiyaga ajratamiz. Komponent hamda huklar ham funksiya boʻlganligi uchun, bu ular uchun ham ishlaydi!

**Qoʻlbola huk nomlanishi "`use`" deb boshlangan JavaScript funksiyasi va u boshqa huklarni chaqira oladi.** Misol uchun, quyidagi `useFriendStatus` bizning birinchi qoʻlbola hukimiz:

```js{3}
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

Buni ichida hech qanday yangilik yoʻq -- mantiqiy qism yuqoridagi komponentlardan koʻchirib olingan. Komponentlarda boʻlganidek, qoʻlbola huklarda ham huklarni shartli asosda chaqirmang.

React kompontentlardan farqli, qoʻlbola huk ayni bir shaklga ega boʻlishi shart emas. Uni aynan qanday argument olishi hamda nima qaytarishini belgilay olamiz. Boshqacha aytganda, u oddiy funksiya kabidir. Bir qarashda u [huklar uchun qoidalar](/docs/hooks-rules.html)ga boʻysunishini anglay olish uchun nomini `use` bilan boshlashingiz shart.

`useFriendStatus` hukining vazifasi doʻstimizning statusiga obuna boʻlish. Shuning uchun argument sifatida `friendID`ni olmoqda va doʻstimiz onlayn yoki yoʻqligini qaytaryapti:

```js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  return isOnline;
}
```

Qani endi qoʻlbola hukimizni qanday ishlatishni oʻrganamiz.

## Qoʻlbola hukni ishlatish {#using-a-custom-hook}

Maqsadimi `FriendStatus` va `FriendListItem` komponentlaridagi bir xil mantiqiy qismlarni yoʻqotmoqchi edik. Ikki komponent ham doʻstimiz onlaynmi yoʻqmi bilmoqchi.

Endi oʻsha mantiqiy qismni `useFriendStatus` hukiga ajratdik va buni *shunchaki ishlata olamiz:*

```js{2}
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Yuklanyapti...';
  }
  return isOnline ? 'Onlayn' : 'Oflayn';
}
```

```js{2}
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

**Bu kod asl misollar bilan bir xilmi?** Ha, u ayni bir xil ravishda ishlaydi. Yaqindan qarasangiz, xatti-harakatga hech qanday oʻzgartirish kiritmadik. Shunchaki ikki funksiyadan ayni bir xil kodni alohida funksiyaga ajratib oldik. **Qoʻlbola huklar Reactʼning imkoniyati emas, balki huklar tuzilishini tabiy ravishda ishlatish usullaridan biri.**

**Qoʻlbola huklarimni nomini “`use`” bilan boshlashim shartmi?** Iltimos, shunday qiling. Bu kelishuv juda muhim. Aks holda, [huklar uchun qoidalar](/docs/hooks-rules.html) buzilmayotganini oʻz-oʻzidan tekshirilmaydi, chunki qaysidir funksiya ichida huklar chaqirilayotganini ayta olmaymiz.

**Do two components using the same Hook share state?** No. Custom Hooks are a mechanism to reuse *stateful logic* (such as setting up a subscription and remembering the current value), but every time you use a custom Hook, all state and effects inside of it are fully isolated.

**Ikki komponent ishlatayotgan hukdagi holat umumiymi?** Yoʻq. Qoʻlbola huklar *holatli mantiqni (stateful logic)* qayta ishlatadigan qurilma hisoblanadi (such as setting up a subscription and remembering the current value), but every time you use a custom Hook, all state and effects inside of it are fully isolated.

**How does a custom Hook get isolated state?** Each *call* to a Hook gets isolated state. Because we call `useFriendStatus` directly, from React's point of view our component just calls `useState` and `useEffect`. And as we [learned](/docs/hooks-state.html#tip-using-multiple-state-variables) [earlier](/docs/hooks-effect.html#tip-use-multiple-effects-to-separate-concerns), we can call `useState` and `useEffect` many times in one component, and they will be completely independent.

### Tip: Pass Information Between Hooks {#tip-pass-information-between-hooks}

Since Hooks are functions, we can pass information between them.

To illustrate this, we'll use another component from our hypothetical chat example. This is a chat message recipient picker that displays whether the currently selected friend is online:

```js{8-9,13}
const friendList = [
  { id: 1, name: 'Phoebe' },
  { id: 2, name: 'Rachel' },
  { id: 3, name: 'Ross' },
];

function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    </>
  );
}
```

We keep the currently chosen friend ID in the `recipientID` state variable, and update it if the user chooses a different friend in the `<select>` picker.

Because the `useState` Hook call gives us the latest value of the `recipientID` state variable, we can pass it to our custom `useFriendStatus` Hook as an argument:

```js
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);
```

This lets us know whether the *currently selected* friend is online. If we pick a different friend and update the `recipientID` state variable, our `useFriendStatus` Hook will unsubscribe from the previously selected friend, and subscribe to the status of the newly selected one.

## `useYourImagination()` {#useyourimagination}

Custom Hooks offer the flexibility of sharing logic that wasn't possible in React components before. You can write custom Hooks that cover a wide range of use cases like form handling, animation, declarative subscriptions, timers, and probably many more we haven't considered. What's more, you can build Hooks that are just as easy to use as React's built-in features.

Try to resist adding abstraction too early. Now that function components can do more, it's likely that the average function component in your codebase will become longer. This is normal -- don't feel like you *have to* immediately split it into Hooks. But we also encourage you to start spotting cases where a custom Hook could hide complex logic behind a simple interface, or help untangle a messy component.

For example, maybe you have a complex component that contains a lot of local state that is managed in an ad-hoc way. `useState` doesn't make centralizing the update logic any easier so you might prefer to write it as a [Redux](https://redux.js.org/) reducer:

```js
function todosReducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, {
        text: action.text,
        completed: false
      }];
    // ... other actions ...
    default:
      return state;
  }
}
```

Reducers are very convenient to test in isolation, and scale to express complex update logic. You can further break them apart into smaller reducers if necessary. However, you might also enjoy the benefits of using React local state, or might not want to install another library.

So what if we could write a `useReducer` Hook that lets us manage the *local* state of our component with a reducer? A simplified version of it might look like this:

```js
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

Now we could use it in our component, and let the reducer drive its state management:

```js{2}
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer, []);

  function handleAddClick(text) {
    dispatch({ type: 'add', text });
  }

  // ...
}
```

The need to manage local state with a reducer in a complex component is common enough that we've built the `useReducer` Hook right into React. You'll find it together with other built-in Hooks in the [Hooks API reference](/docs/hooks-reference.html).
