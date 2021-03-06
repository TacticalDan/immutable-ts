
The simple Typescript'y implementation of ImmutableJS --- https://github.com/immutable-js/immutable-js

``` typescript
// Immutable type: Just a recursive version of the pre-existing Readonly<T> type built-in to typescript
export type Immutable<T> = {
    readonly [P in keyof T]: Immutable<T[P]>;
};
// 👍 That's it!
```

``` typescript
// ⏰ Example time! Here's a mutable type definition
type MutableState = {
    ink: string,
    metroFeud: {
        stamps: Array<{
            ozone: number,
        }>,
    },
};

// Can be used like normal, able to reach deep within the object
// and change its internals without anybody knowing 👎
const stateA: MutableState = {
    ink: "hi",
    metroFeud: { stamps: [{ozone: 1}] },
};
stateA.metroFeud.stamps[0].ozone = 2;
// ☁ We're assuming some other code looking for changes to stateA will recurse into this object
// and check to see if the state is changed,
// or bear the responsibility of notifying all code from here? 😔

// 💡 Defining an Immutable version of MutableThing
type ImmutableState = Immutable<MutableState>;

// Can be initialized the exact same!
let stateB: ImmutableState = {
    ink: "hi",
    metroFeud: { stamps: [{ozone: 1}] },
};
// ✨ stateB is now an immutable object, 'stamp' is a ReadonlyArray that can't be modified as well 👍
// 💻 stateB.metroFeud.stamps[0].ozone = 2; // prevented. That's sick 🤘
// 💻 stateB.metroFeud.stamps.push({ ozone: 2 }; // also illegal because 'stamps' is now a ReadonlyArray<T>

// So stateB is safe from being mutated deeply inside of it, but stateB itself can still be updated:
const nextStateB = {
    ...stateB, metroFeud: {
        ...stateB.metroFeud, stamps: [
            ...stateB.metroFeud.stamps, { ozone: 2 }]}};

// Equivalent of stateB.metroFeud.stamp.push({ozone: 2}) but made immutable!
// It means we can have less state variables [less chaos, bugs],
// because each variable can contain more meaning.

// You know something's changed since before because you can do this:
if (stateB != nextStateB) {
    console.log("🚨 Something changed! ⚡ Update the views!");
}
// Instead of having to loop though each member and element
// of stateB AND thingB to determine if a change has occured.

// Overall, way easier to work with compared to ImmutableJS.
```

``` typescript
// Turns out in complex codebases, recursive type definitions and generics don't cooperate well, as I started getting this fun bug ---
// https://github.com/microsoft/TypeScript/issues/34933
// It is better in this case to just define your original types with extra 'readonly' syntax for each member
// which works just as well, just some extra visual noise 🌊
type ImmutableState = {
    readonly ink: string,
    readonly metroFeud: {
        readonly stamps: ReadonlyArray<{
            readonly ozone: number,
        }>,
    },
};
