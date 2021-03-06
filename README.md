# react16 源码研究

## 流程图
![image](https://raw.githubusercontent.com/LeoHuiyi/react16-source-code-study/master/react16-leo.png)


## 主要类型
```javaScript
TypeOfSideEffect = {
    // Don't change these two values:
    NoEffect: 0, //           0b00000000
    PerformedWork: 1, //      0b00000001
    // You can change the rest (and add more).
    Placement: 2, //          0b00000010
    Update: 4, //             0b00000100
    PlacementAndUpdate: 6, // 0b00000110
    Deletion: 8, //           0b00001000
    ContentReset: 16, //      0b00010000
    Callback: 32, //          0b00100000
    Err: 64, //               0b01000000
    Ref: 128 //              0b10000000
}


TypeOfInternalContext = {
    NoContext: 0,
    AsyncUpdates: 1,
}


PriorityLevel = {
    NoWork: 0, // No work is pending.
    SynchronousPriority: 1, // For controlled text inputs. Synchronous side-effects.
    TaskPriority: 2, // Completes at the end of the current tick.
    HighPriority: 3, // Interaction that needs to complete pretty soon to feel responsive.
    LowPriority: 4, // Data fetching, or result from updating stores.
    OffscreenPriority: 5 // Won't be visible but do the work in case it becomes visible.
}

Update = {
    priorityLevel: PriorityLevel | null,
    expirationTime: ExpirationTime,
    partialState: PartialState<any, any>,
    callback: Callback | null,
    isReplace: boolean,
    isForced: boolean,
    isTopLevelUnmount: boolean,
    next: Update | null,
}

UpdateQueue = {
    first: Update | null,
    last: Update | null,
    hasForceUpdate: boolean,
    callbackList: null | Array<Callback>,
    
    // Dev only
    isProcessing?: boolean,
}

TypeOfWork = {
    IndeterminateComponent: 0, // Before we know whether it is functional or class
    FunctionalComponent: 1,
    ClassComponent: 2,
    HostRoot: 3, // Root of a host tree. Could be nested inside another node.
    HostPortal: 4, // A subtree. Could be an entry point to a different renderer.
    HostComponent: 5,
    HostText: 6,
    CoroutineComponent: 7,
    CoroutineHandlerPhase: 8,
    YieldComponent: 9,
    Fragment: 10
}

type Fiber = {|
    // These first fields are conceptually members of an Instance. This used to
    // be split into a separate type and intersected with the other Fiber fields,
    // but until Flow fixes its intersection bugs, we've merged them into a
    // single type.

    // An Instance is shared between all versions of a component. We can easily
    // break this out into a separate object to avoid copying so much to the
    // alternate versions of the tree. We put this on a single object for now to
    // minimize the number of objects created during the initial render.

    // Tag identifying the type of fiber.
    tag: TypeOfWork,

    // Unique identifier of this child.
    key: null | string,

    // The function/class/module associated with this fiber.
    type: any,

    // The local state associated with this fiber.
    stateNode: any,

    // Conceptual aliases
    // parent : Instance -> return The parent happens to be the same as the
    // return fiber since we've merged the fiber and instance.

    // Remaining fields belong to Fiber

    // The Fiber to return to after finishing processing this one.
    // This is effectively the parent, but there can be multiple parents (two)
    // so this is only the parent of the thing we're currently processing.
    // It is conceptually the same as the return address of a stack frame.
    return: Fiber | null,

    // Singly Linked List Tree Structure.
    child: Fiber | null,
    sibling: Fiber | null,
    index: number,

    // The ref last used to attach this node.
    // I'll avoid adding an owner field for prod and model that as functions.
    ref: null | (((handle: mixed) => void) & { _stringRef: ?string }),

    // Input is the data coming into process this fiber. Arguments. Props.
    pendingProps: any, // This type will be more specific once we overload the tag.
    memoizedProps: any, // The props used to create the output.

    // A queue of state updates and callbacks.
    updateQueue: UpdateQueue | null,

    // The state used to create the output
    memoizedState: any,

    // Bitfield that describes properties about the fiber and its subtree. E.g.
    // the AsyncUpdates flag indicates whether the subtree should be async-by-
    // default. When a fiber is created, it inherits the internalContextTag of its
    // parent. Additional flags can be set at creation time, but after than the
    // value should remain unchanged throughout the fiber's lifetime, particularly
    // before its child fibers are created.
    internalContextTag: TypeOfInternalContext,

    // Effect
    effectTag: TypeOfSideEffect,

    // Singly linked list fast path to the next fiber with side-effects.
    nextEffect: Fiber | null,

    // The first and last fiber with side-effect within this subtree. This allows
    // us to reuse a slice of the linked list when we reuse the work done within
    // this fiber.
    firstEffect: Fiber | null,
    lastEffect: Fiber | null,

    // Represents a time in the future by which this work should be completed.
    // This is also used to quickly determine if a subtree has no pending changes.
    expirationTime: ExpirationTime,

    // This is a pooled version of a Fiber. Every fiber that gets updated will
    // eventually have a pair. There are cases when we can clean up pairs to save
    // memory if we need to.
    alternate: Fiber | null,

    // Conceptual aliases
    // workInProgress : Fiber ->  alternate The alternate used for reuse happens
    // to be the same as work in progress.
    // __DEV__ only
    _debugID?: number,
    _debugSource?: Source | null,
    _debugOwner?: Fiber | null,
    _debugIsCurrentlyTiming?: boolean
|}
```
