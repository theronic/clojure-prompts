---
description: Electric Clojure v3
globs: "*.cljc"
alwaysApply: false
---
# Electric Clojure v3: A Comprehensive Style Guide

## Introduction

Electric Clojure is a reactive Clojure(Script) dialect for building web applications with a unified client-server programming model. This guide distills best practices, patterns, and idioms for writing idiomatic, efficient Electric Clojure v3 applications. It emphasizes Electric's core strengths: reactive computation, seamless client-server transfer, and efficient UI rendering.

Electric Clojure's design enables developers to write full-stack applications in a single language with a unified mental model, eliminating the typical boundaries between client and server. This guide will help you harness these strengths to produce robust, maintainable Electric Clojure code.

## 1. Electric Clojure Philosophy and Fundamentals

### 1.1 Core Principles

#### Reactivity

Electric is fundamentally a reactive programming system, where changes to data automatically propagate through the application.

- **Values flowing through a DAG**: Electric programs are directed acyclic graphs (DAGs) of computations, with values flowing from sources to sinks.
- **Differential updates**: Only recompute what has changed, with fine-grained reactivity.
- **Signals, not streams**: Electric models signals (like mouse coordinates) rather than streams (like mouse clicks).
- **Work-skipping**: If an argument to a function hasn't changed, the function doesn't need to be recomputed.
- **Backpressure**: Electric automatically handles backpressure throughout the application, ensuring components aren't overwhelmed with data.

```clojure
;; A reactive timer that automatically updates
(e/defn Timer []
  (let [seconds (/ (e/System-time-ms) 1000)]
    (dom/div (dom/text "Time: " seconds))))
```

#### Client-Server Transfer

Electric allows for seamless client-server programming within the same codebase.

- **Transfer boundaries are explicit**: Use `e/server` and `e/client` to specify where code runs.
- **Values transfer, references don't**: Only serializable values can cross network boundaries.
- **Platform interop forces site**: Interop with platform-specific code (like DOM or Java) forces expressions to a specific site.
- **Edges aren't sited**: Shared bindings pass through function calls symbolically, not forcing transfer.
- **Dynamic siting**: Site inheritance follows dynamic scope in v3.

```clojure
;; Server-side query with client-side rendering
(e/defn UserProfile [user-id]
  (let [user-data (e/server (db/get-user user-id))]
    (e/client
      (dom/div
        (dom/h1 (dom/text (:name user-data)))
        (dom/p (dom/text (:bio user-data)))))))
```

#### Differential Collections

To send only changes over the wire, the Electric `e/for` looping construct expects a *differential collection*, calculated via `(e/diff-by predicate collection)`.

`(e/for-by :id [uid user-ids] ...)` is equivalent to `(e/for [uid (e/diff-by :id user-ids)] ...)`.

Prefer to name differential collections explicitly by prefixing with `diffed-`, e.g. `diffed-users`. 

#### Unified Programming Model

Electric unifies client and server code into a single program and mental model.

- **Single language across stack**: Write your entire application in Clojure(Script).
- **Shared syntax and semantics**: Same language constructs work on both client and server.
- **Automatic serialization**: Electric handles value transfer between environments.
- **Single-pass rendering**: Traverse and render server data directly to client DOM in one pass.
- **Bidirectional reactivity**: Changes flow in both directions through the DAG.

```clojure
;; Combining server data access with client input in one function
(e/defn SearchUsers []
  (let [search-term (e/client (dom/input (dom/On "input" #(-> % .-target .-value) "")))
        results (e/server 
                  (when (seq search-term)
                    (db/search-users search-term)))]
    (e/client
      (dom/div
        (dom/h2 (dom/text "Search Results"))
        (e/for [user results]
          (dom/div (dom/text (:name user))))))))
```

### 1.2 Electric vs. Traditional Clojure

Electric Clojure extends regular Clojure with reactive semantics, which creates some important differences:

- **Reactivity as default**: Every expression is potentially reactive, recalculating when inputs change.
- **Concurrent evaluation**: Expressions in different sites run concurrently, not sequentially.
- **Tables (amb) vs. collections**: Electric uses tables (amb values) as its primary collection abstraction.
- **Differential evaluation**: Functions are called on changes to their inputs, not just on initial evaluation.
- **Reactive lifecycles**: Components have mount/unmount lifecycle tied to their place in the DAG.
- **Transfer semantics**: Need to reason about what moves between client and server.

```clojure
;; In regular Clojure, this would print once
;; In Electric, it prints whenever x changes
(let [x (e/watch !state)]
  (println x))
```

## 2. Code Organization and Structure

### 2.1 Namespace Organization

1. **One component per namespace**: For complex components, define them in their own namespace.

2. **Group related components**: Keep related components in the same namespace.

3. **Reuse ns conventions**: Follow standard Clojure namespace conventions.

4. **Separate client/server only code**: Code that runs exclusively on one platform should be in separate namespaces with appropriate reader conditionals.

5. **Use cljc for shared code**: Write most Electric code in .cljc files for cross-platform compatibility.

```clojure
(ns my-app.user-profile
  (:require [hyperfiddle.electric3 :as e]
            [hyperfiddle.electric-dom3 :as dom]
            #?(:clj [my-app.db :as db]) ; server-side only dependency
            #?(:cljs [my-app.ui.client :as ui-client]) ; client-side only dependency
            [my-app.ui.components :as ui])) ; shared UI components
```

### 2.2 Component Structure

1. **Single responsibility**: Components should have a clear, focused purpose.

2. **Explicit transfer boundaries**: Make client/server boundaries clear and deliberate.

3. **Keep components small**: Break large components into smaller, composable pieces.

4. **Clear input/output contract**: Define what data flows in and out of components.

5. **Pure rendering**: Keep rendering logic separate from data fetching when possible.

```clojure
;; Good: component with clear responsibilities
(e/defn UserCard [db user-id]
  (let [user (e/server (db/get-user db user-id))]
    (e/client
      (dom/div (dom/props {:class "user-card"})
        (dom/h2 (dom/text (:name user)))
        (dom/p (dom/text (:email user)))))))

;; Better: separated data and UI components
(e/defn UserData [db user-id]
  (e/server (db/get-user db user-id)))

(e/defn UserCard [db user-id]
  (let [user (UserData db user-id)]
    (dom/div (dom/props {:class "user-card"})
      (dom/h2 (dom/text (:name user)))
      (dom/p (dom/text (:email user))))))
```

### 2.3 State Management

1. **Server state in atoms**: Use atoms for shared server-side state.

2. **Reactive state with e/watch**: Connect atoms to the reactive system with `e/watch`.

3. **Optimize state granularity**: Balance between too many small atoms and too few large atoms.

4. **Shared vs. local state**: Decide deliberately what state should be global vs. component-local.

5. **Client-side state**: Use atoms for UI-only state that doesn't need server persistence.

```clojure
;; Server-side shared state
#?(:clj (defonce !app-state (atom {:users {} :settings {}})))

;; Reactive component using shared state
(e/defn UserList []
  (e/server
    (let [users        (:users (e/watch !app-state))
          diffed-users (e/diff-by :id users)]
     (e/client
       (dom/div
         (e/for [user diffed-users]
           (UserCard (:id user))))))))
```

## 3. Reactive Programming Patterns

### 3.1 Working with Electric Tables and e/amb

Electric tables (created with `e/amb`) are fundamental to Electric's reactive model:

- **Use e/amb for concurrent values**: Tables hold multiple values in superposition.
- **Flattening semantics**: Nested `e/amb` calls flatten automatically.
- **Auto-mapping of functions**: Functions automatically map over values in tables.
- **Use e/for for iteration**: When working with tables, use `e/for` for proper isolation.
- **Materialization with e/as-vec**: Convert Electric tables to Clojure vectors with `e/as-vec`.
- **Understand cartesian products**: Be careful with operations that create cross products of tables.

```clojure
;; Creating a table with multiple values
(e/amb 1 2 3) ;; Table with three values in superposition

;; Auto-mapping
(inc (e/amb 1 2 3)) ;; Results in (e/amb 2 3 4)

;; Using e/for for iteration and isolation
(e/for [x (e/amb 1 2 3)]
  (dom/div (dom/text x))) ;; Creates three separate divs

;; Converting to a regular vector
(e/as-vec (e/amb 1 2 3)) ;; [1 2 3]
```

### 3.2 Reactive Flow Control

Control flow in Electric requires special consideration due to reactivity:

- **Use e/for for collection iteration**: Ensures proper lifecycle management for items.
- **Prefer e/for over if with tables**: `e/for` provides better semantics with non-singular values.
- **Use e/for-by for keyed collections**: When items have stable identities, use `e/for-by`.
- **Use e/when for conditional rendering**: For simple cases with a single condition.
- **Careful with product semantics**: Be aware of unintended product semantics with if/when and tables.

```clojure
;; Conditional rendering with e/when
(e/when logged-in?
  (UserDashboard db user-id))

;; Iteration with stable identity
(e/for-by :id [user users]
  (UserCard db user))

;; Avoiding cartesian products with if
(e/for [x (e/amb 1 2 3)]
  (e/when (odd? x)
    (dom/div (dom/text x))))
```

### 3.3 Working with Events

Events in Electric are handled through tokens and event flow:

- **Use dom/On for simple events**: Captures DOM events and their values.
- **Use dom/On-all for concurrent events**: Handles multiple events without waiting for previous ones to complete.
- **Token-based event tracking**: Events return `[t v]` pairs for lifecycle tracking.
- **Command patterns for state changes**: Use tokens to manage command state like pending/complete.

```clojure
;; Simple event handling
(let [value (dom/input (dom/On "input" #(-> % .-target .-value) ""))]
  (dom/div (dom/text "You typed: " value)))

;; Concurrent events with command tracking
(e/for [[t e] (dom/On-all "click")]
  (let [res (e/server (process-click!))]
    (case res
      ::ok (t) ;; Mark command as completed
      ::error (prn "Error!"))))
```

### 3.4 Managing Side Effects

Side effects require careful handling in a reactive system:

- **Use e/on-unmount for cleanup**: Register cleanup functions when components unmount.
- **Mark effects with !**: Use `!` suffix for functions with side effects.
- **Isolate effects from pure logic**: Keep side effects separate from pure computation.
- **Use e/Offload for blocking operations**: Run blocking operations without stalling the reactive system.

```clojure
;; Component with cleanup
(e/defn ResourceComponent []
  (let [resource (e/server (open-resource!))]
    (e/on-unmount #(e/server (close-resource! resource)))
    (dom/div (dom/text "Using resource: " (resource-name resource)))))

;; Offloading blocking work
(e/defn SlowOperation []
  (let [result (e/server (e/Offload #(do-expensive-calculation)))]
    (dom/div (dom/text "Result: " result))))
```

## 4. Client-Server Patterns

### 4.1 Transfer Boundaries

Understanding and managing transfer boundaries is crucial in Electric:

- **Platform interop forces site**: Interop with platform-specific code forces expressions to a specific site.
- **Values transfer, references don't**: Only serializable values can cross network boundaries.
- **Site determines where code runs**: The site of an expression determines where it executes.
- **Dynamic site inheritance**: Site is inherited through dynamic scope.
- **DOM operations are auto-sited**: Electric DOM macros auto-site their contents to the client.

```clojure
;; Server interop forces server site
(e/server (java.io.File. "example.txt")) 

;; Client interop forces client site
(e/client (js/alert "Hello"))

;; Value transfers across boundary
(e/client (dom/div (dom/text (e/server (db/get-username)))))

;; Reference doesn't transfer
(e/server (let [conn (db/connect!)] ;; conn stays on server
            (e/client (dom/div (dom/text "Connected")))))
```

### 4.2 Optimizing Network Traffic

Minimizing unnecessary network transfers:

- **Batch related queries**: Fetch related data in one operation.
- **Use e/server sparingly**: Don't create unnecessary round trips.
- **Transfer minimal data**: Only send what's needed across the network.
- **Use client-side filtering**: When possible, filter large datasets on the server.
- **Watch for accidental transfers**: Be careful about unintended data movement.

```clojure
;; Bad: Multiple round trips
(e/defn UserProfile [db user-id]
  (e/server
    (let [user    (db/get-user db user-id)
          posts   (db/get-user-posts db user-id) ;; Separate trip
          friends (db/get-user-friends db user-id)] ;; Separate trip
      (e/client (RenderUserProfile user posts friends)))))

;; Better: Batched query
(e/defn UserProfile [db user-id]
  (let [{:as user-data :keys [user posts friends]}
        (e/server (db/get-user-with-posts-and-friends db user-id))]
    (RenderUserProfile user posts friends)))
```

### 4.3 Authentication and Security

Security considerations for Electric applications:

- **Validate on server**: Always validate user input on the server.
- **Never trust client data**: All client-provided data must be validated.
- **Use server-side permissions**: Check permissions on the server before operations.
- **Secure sensitive operations**: Place security-sensitive code in `e/server` blocks.
- **Expose only necessary data**: Only send the client data it needs.

```clojure
;; Secure server-side validation
(e/defn UpdateUser [user-id user-data]
  (e/server
    (when (authorized? user-id) ;; Server-side permission check
      (when (valid-user-data? user-data) ;; Server-side validation
        (db/update-user! conn user-id user-data)))))

;; Only sending necessary data to client
(e/defn UserProfile [db user-id]
  (let [user (e/server 
               (-> (db/get-user db user-id)
                   (select-keys [:name :bio :public-info]))) ;; Exclude sensitive data
    (RenderUserProfile user)))
```

## 5. UI Patterns and Best Practices

### 5.1 DOM Manipulation

Effective DOM manipulation in Electric:

- **Use electric-dom3 macros**: Leverage the reactive DOM API.
- **Props for attributes**: Use `dom/props` for element attributes.
- **Dynamic text with dom/text**: Create reactive text nodes.
- **Event handling with dom/On**: Capture DOM events reactively.
- **Element reference with dom/node**: Access the DOM element for imperative operations.

```clojure
;; Complete DOM example
(e/defn Button [label on-click]
  (dom/button
    (dom/props {:class "btn btn-primary"
                :disabled (nil? on-click)})
    (dom/On "click" (fn [e] 
                      (.preventDefault e)
                      (when on-click (on-click))))
    (dom/text label)))
```

### 5.2 Forms and User Input

Building interactive forms in Electric:

- **Controlled inputs**: Use reactive values to control input state.
- **Form validation**: Validate forms on both client and server.
- **Input debouncing**: Debounce rapidly changing inputs when necessary.
- **Multi-step forms**: Break complex forms into manageable stages.
- **Optimistic updates**: Provide immediate feedback before server confirmation.

```clojure
;; Simple controlled input
(e/defn InputField [label value on-change]
  (dom/div
    (dom/label (dom/text label))
    (dom/input
      (dom/props {:value value
                 :type "text"})
      (dom/On "input" #(on-change (-> % .-target .-value))))))

;; Form with validation
(e/defn LoginForm []
  (let [!email    (atom "")
        !password (atom "")
        errors (e/server
                 (cond-> {}
                   (not (valid-email? (e/watch !email)))
                   (assoc :email "Invalid email")
                   
                   (< (count (e/watch !password)) 8)
                   (assoc :password "Password too short")))]
    (dom/form
      (InputField "Email" (e/watch !email) #(reset! !email %))
      (when (:email errors)
        (dom/div (dom/props {:class "error"}) (dom/text (:email errors))))
      
      (InputField "Password" (e/watch !password) #(reset! !password %))
      (when (:password errors)
        (dom/div (dom/props {:class "error"}) (dom/text (:password errors))))
      
      (dom/button
        (dom/props {:type "submit"
                   :disabled (not-empty errors)})
        (dom/text "Login")))))
```

### 5.3 List Rendering and Collection Handling

Efficiently rendering collections:

- **Use e/for-by for keyed collections**: Provide stable identity for efficient updates.
- **Process collections server-side**: Sort, filter, and limit collections on the server when possible.
- **Virtual rendering for large lists**: Consider virtual rendering for very large collections.
- **Paginated loading**: Implement pagination for large datasets.
- **Optimistic collection updates**: Update collections optimistically for better UX.

```clojure
;; Efficient list rendering with keyed iteration
(e/defn UserList [users]
  (dom/ul
    (e/for-by :id [user users]
      (dom/li
        (dom/text (:name user))))))

;; Server-side processing before rendering
(e/defn SearchResults [db query]
  (let [results (e/server
                  (-> (db/search db query)
                      (sort-by :relevance)
                      (take 20)))]
    (dom/div
      (dom/h2 (dom/text "Results for: " query))
      (UserList results))))
```

### 5.4 Performance Optimization

Techniques for optimizing Electric UI performance:

- **Minimize state changes**: Only update state when necessary.
- **Use e/memo for expensive calculations**: Memoize results of expensive computations.
- **Control reactivity granularity**: Balance between too fine and too coarse reactivity.
- **Implement virtualization**: Use virtual rendering for large lists.
- **Profile and optimize**: Use browser dev tools to identify bottlenecks.

```clojure
;; Memoizing expensive calculations
(e/defn ExpensiveCalculation [data]
  (e/memo
    (slow-calculation data)))

;; Virtualized list rendering (simplified)
(e/defn VirtualList [items]
  (let [visible-range (calculate-visible-range)]
    (dom/div
      (dom/props {:style {:height (str (* (count items) item-height) "px")}})
      (e/for-by :id [item (subvec items (:start visible-range) (:end visible-range))]
        (dom/div
          (dom/props {:style {:position "absolute"
                             :top (str (* (:index item) item-height) "px")}})
          (RenderItem item))))))
```

## 6. Advanced Patterns

### 6.1 Optimistic Updates

Implementing responsive UIs with optimistic updates:

- **Apply changes locally first**: Update UI before server confirmation.
- **Track in-flight commands**: Use tokens to track command state.
- **Reconcile server responses**: Merge server updates with optimistic ones.
- **Handle failures gracefully**: Provide visual feedback for failed operations.
- **Use with-cycle for edit loops**: Loop in-flight edits through the component.

```clojure
;; Optimistic todo creation
(e/defn TodoList []
  (let [server-todos (e/server (e/watch !todos))
        !local-edits (atom {})
        all-todos (concat server-todos 
                          (vals (e/watch !local-edits)))]
    (dom/div
      (e/for-by :id [todo all-todos]
        (TodoItem todo))
      (dom/form
        (dom/input (dom/On "submit" 
          (fn [e]
            (.preventDefault e)
            (let [value (-> e .-target .-elements .-todo .-value)
                  temp-id (random-uuid)
                  temp-todo {:id temp-id :text value :status :pending}]
              ;; Add optimistic todo
              (swap! !local-edits assoc temp-id temp-todo)
              ;; Send to server
              (e/server
                (let [result (db/add-todo! value)]
                  ;; On success, remove temporary todo
                  (e/client
                    (swap! !local-edits dissoc temp-id)))))))))))))
```

### 6.2 Error Handling

Robust error handling in Electric applications:

- *try..catch is not yet supported in Electric v3. You can only `try..catch` in Clojure-land functions.
- **Provide fallback UI**: Show useful fallback content when errors occur.
- **Log errors server-side**: Record errors for debugging.
- **Distinguish between different error types**: Handle different errors appropriately.
- **Retry mechanisms**: Implement retry logic for transient failures.

### 6.3 Loading States and Suspense

Managing loading states effectively:

- **Show loading indicators**: Provide visual feedback during loading.
- **Use progressive loading**: Load and display content incrementally.
- **Skeleton screens**: Show skeleton UI while content loads.
- **Control loading granularity**: Balance between fine and coarse-grained loading states.
- **Loading timeouts**: Consider timeouts for loading indicators.

```clojure
;; Component with loading state
(e/defn AsyncContent [db query]
  (let [!loading? (atom true)
        result (e/server
                 (try
                   (let [data (slow-query db query)]
                     (e/client (reset! !loading? false))
                     data)
                   (catch Exception e
                     (e/client (reset! !loading? false))
                     nil)))]
    (e/if (e/watch !loading?)
      (LoadingSpinner)
      (if result
        (ContentView result)
        (ErrorMessage)))))
```

### 6.4 Custom Reactivity

Working with custom reactive patterns:

- **Custom event sources**: Create reactive sources from external events.
- **Manual reactivity with e/watch/e/write!**: Direct control when needed.
- **Interop with external reactive systems**: Adapter patterns for external reactivity.
- **Custom differential collections**: Build custom reactive collection abstractions.
- **Controlled reactivity flow**: Fine-tune when and how reactivity propagates.

```clojure
;; Creating a reactive source from external events
(e/defn ExternalEventSource []
  (let [!value (atom nil)]
    ;; Connect to external event source
    (js/externalAPI.onEvent #(reset! !value %))
    (e/on-unmount #(js/externalAPI.removeEventListener))
    
    ;; Return reactive value
    (e/watch !value)))

;; Using the external event source
(e/defn ExternalEventConsumer []
  (let [event-value (ExternalEventSource)]
    (dom/div (dom/text "Latest event: " event-value))))
```

## 7. Testing and Debugging

### 7.1 Testing Electric Components

Strategies for testing Electric applications:

- **Unit test pure functions**: Extract and test pure logic separately.
- **Mock external dependencies**: Isolate components by mocking dependencies.
- **Test server components**: Test server-side logic with regular Clojure testing tools.
- **Test client components**: Use ClojureScript testing tools for client components.
- **Integration testing**: Test full client-server flows when necessary.

```clojure
;; Extracting pure functions for testing
(defn calculate-total [items]
  (reduce + (map :price items)))

;; Using in Electric component
(e/defn ShoppingCart [items]
  (let [total (calculate-total items)]
    (dom/div
      (dom/h2 (dom/text "Shopping Cart"))
      (e/for-by :id [item items]
        (CartItem item))
      (dom/div (dom/text "Total: $" total)))))

;; Test the pure function
(deftest calculate-total-test
  (is (= 100 (calculate-total [{:price 50} {:price 50}])))
  (is (= 0 (calculate-total []))))
```

### 7.2 Debugging Techniques

Effective debugging for Electric applications:

- **Use println/prn strategically**: Place print statements for visibility. These will only eval if their values changed.
- **Inspect reactive values**: Use `(prn (e/as-vec reactive-value))` for readable debug output.
- **Check transfer boundaries**: Confirm where code is running.
- **Browser DevTools**: Use browser tools for client-side debugging.
- **Server logs**: Check server logs for server-side issues.
- **Isolate components**: Debug issues by isolating components.

```clojure
;; Debug output for reactive values
(e/defn DebugComponent [data]
  (prn "Data:" (e/as-vec data))
  (dom/div
    (e/for [item data]
      (do
        (prn "Rendering item:" item)
        (dom/div (dom/text (str item)))))))
```

## 8. Deployment and Production

### 8.1 Building for Production

Preparing Electric applications for production:

- **Optimize client assets**: Minimize and bundle client-side code.
- **Configure caching**: Set appropriate cache headers.
- **Error tracking**: Implement error tracking and monitoring.
- **Performance monitoring**: Add performance measurement tools.
- **Version management**: Ensure client and server versions match.

```clojure
;; Production entrypoint example
(e/defn ProdApp []
  (e/client
    (dom/div
      (dom/props {:class "app-container"})
      (Header)
      (MainContent)
      (Footer))))
```

### 8.2 Server Configuration

Configuring servers for Electric applications:

- **Websocket setup**: Configure proper websocket support.
- **Resource allocation**: Allocate appropriate resources based on expected load.
- **Session management**: Implement proper session handling.
- **Security headers**: Configure security headers for the application.
- **Load balancing**: Set up load balancing if needed.

## 9. Idioms and Conventions

### 9.1 Naming and Style

Consistent naming and style practices:

- **Use e/defn compiler macro for Electric functions**: Clearly indicate Electric functions.
- **Use ! prefix for stateful values like atoms**: Indicate reference types with `!`, e.g. `!counter`.
- **Use ! suffix for mutation functions**: Mark functions with side effects, e.g. `save-user!`.
- **Use ? suffix for predicates**: Follow standard Clojure conventions.
- **Clear, descriptive component names**: Use specific, meaningful names.

```clojure
;; Good naming examples
#?(:clj (defonce !app-state (atom {}))) ;; Atom with ! prefix

(e/defn UserProfile [user-id]) ;; Clear component name

(defn valid-email? [email]) ;; Predicate with ? suffix

(defn save-user! [user]) ;; Mutation with ! suffix
```

### 9.2 Common Patterns and Idioms

Recurring patterns in Electric code:

- **e/for-by for keyed collections**: Always use keys for stable identity.
- **e/server/e/client for transfer boundaries**: Keep boundaries clear and explicit.
- **e/watch for connecting atoms**: Standard way to make atoms reactive.
- **e/watch + UI = controlled component**: Pattern for controlled inputs.
- **e/on-unmount for cleanup**: Standard cleanup mechanism.

```clojure
;; Common idioms
(e/defn DataList [items]
  (dom/ul
    (e/for-by :id [item items] ;; Keyed iteration
      (dom/li (dom/text (:name item))))))

(e/defn StateComponent []
  (let [state (e/watch !state)] ;; Connecting atom to reactivity
    (dom/div (dom/text "State: " state))))

(e/defn CleanupComponent []
  (let [resource (init-resource!)]
    (e/on-unmount #(cleanup-resource! resource)) ;; Cleanup on unmount
    (dom/div (dom/text "Resource: " resource))))
```

## Conclusion

Electric Clojure v3 enables a powerful new paradigm for web application development, unifying client and server into a single programming model. By following the principles and patterns in this guide, you can create efficient, maintainable applications that leverage Electric's strengths.

Remember that Electric is still evolving, and best practices may change as the system matures. Stay connected with the Electric community to keep up with the latest developments and patterns.