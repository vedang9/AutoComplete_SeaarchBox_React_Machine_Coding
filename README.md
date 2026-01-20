# AutoComplete_SeaarchBox_React_Machine_Coding
An Auto-Complete search box to show relevent matching results with caching and de-bounce implemented while calling the API.

## Performance & Browser Optimisations
* ✔ Debouncing reduces network traffic
* ✔ AbortController avoids race conditions
* ✔ useRef cache prevents unnecessary re-renders

## Caching
To store cache, useRef is used instead of useState.   
If useState would have been used (eg- const [cache, setCache] = useState({});) then updating the cache state variable would have ccause re-rendering of the UI as any state update triggers a re-render of the component.   
So everytime we would have updated the state using setCache(...), we would have triggered 
  * a render
  * reconciliation
  * diffing  
For no UI benefit  
This is called an unnecessary render. <br/><br/>
But we are not using cache to in rendering the UI. The 'results' state variable is being used to render the search results in the UI. So even if cache updates, not re-rendering the component again will optimise the appliction.

### How useRef solves this
What useRef actually does <br/>
````const cache = useRef({});````
It Creates an object like:
````cache.current = {}```` <br/>
Critical difference:  
Updating ref.current does NOT trigger a re-render  
<b>React does not track refs for rendering.</b>

### Mental Model
<b>State</b> ````(useState)````
* Tracked by React
* Used to drive UI
* Triggers re-render

<b>Ref</b> ````(useRef)````
* Tracked by you
* Used for mutable values
* Does NOT affect UI
* Does NOT re-render

## Abort Controller

The problem it solves <b>(race condition)</b>

Imagine user types fast:

User types "p"
→ API request sent

User types "pa"
→ API request sent

If "p" request responds after "pa"

❌ Old results overwrite new results
❌ UI shows wrong suggestions

This is called a race condition.

### What is AbortController?
AbortController is a browser API that allows you to:
* Cancel an ongoing asynchronous task (like fetch)
* Signal that the request is no longer needed

````
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort();
````

### How it’s used in code
```` controllerRef.current?.abort(); ```` <br/>
Meaning if there is an existing request <b>Cancel it immediately</b>

Then
````controllerRef.current = new AbortController();```` <br>
It means create a new controller. This controller is tied to the new request <br/>

````
const res = await fetch(API_URL + query, {
  signal: controllerRef.current.signal,
});
```` 
Here, fetch listens to the abort signal. If aborted → throws an AbortError </br>

#### Error Handling
````
catch (err) {
  if (err.name !== "AbortError") {
    console.error(err);
  }
}
````
Why this check exists:  
When we get Abort Error,  we don’t want to log them as real errors  
✔ Clean error handling
