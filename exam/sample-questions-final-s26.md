# FTEC 6V97 – Financial Applications of Web Development

## Sample Exam — Study Reference

> **This sample exam is provided so that you can familiarize yourself with the format, style, and depth of questions you will encounter on the final exam. It is shorter than the actual exam and covers a representative selection of topics. Answers are included at the end of each section.**

---

## Part I — Multiple Choice

Select the best answer for each question.

1. Which of the following correctly describes how JavaScript treats the comparison '5' === 5?

- a) true — both operands are coerced to the same type before comparison
- b) true — the values are equal
- c) false — the operands are different types and === does not coerce
- d) Throws a TypeError

2. An element has both class="highlight" and id="banner". The stylesheet contains:

```css
.highlight { color: orange; }
#banner    { color: blue;   }
```

What color is the element's text?

- a) orange — class rules win because they are more general
- b) blue — IDs have higher specificity than classes
- c) The color of whichever rule appears later in the file
- d) A blend of orange and blue

3. What does useEffect(() => { fetch('/api/data') }, [userId]) do?

- a) Runs the fetch once when the component first mounts and never again
- b) Runs the fetch after every render of the component
- c) Runs the fetch on mount, and again any time userId changes
- d) Runs the fetch only when the component unmounts

4. A developer wants a Pydantic field that is OPTIONAL and defaults to None. Which line is correct?

- a) name: str = Field(...)
- b) name: Optional[str] = None
- c) name: Optional[str] = Field(...)
- d) name: str

## Part II — Code Evaluation

1. Predict the output and answer the questions:

```javascript
const tickers = ['aapl', 'msft', 'goog'];const upper = tickers.map(t => t.toUpperCase());console.log(upper);console.log(tickers);console.log(tickers === upper);
```

- a) What does each console.log print?
- b) Why does the third console.log return what it does?

2. Read the component and answer the questions:

```javascript
function Cart() {  
    const [items, setItems] = useState([]);  
    
    function addItem(name) {    
        items.push(name);          // line A 
        setItems(items);           // line B  
    } 
    
    return (<ul>{items.map(i => <li key={i}>{i}</li>)}</ul>);}
```

- a) When the user clicks a button that calls addItem('AAPL'), the items array is technically updated, but the ul element on screen does not change. Why does React fail to re-render?
- b) Rewrite addItem so the UI updates correctly.

3. Examine the route below:

```python
from pydantic import BaseModel, Field
class CreateOrderSchema(BaseModel):    
    ticker: str = Field(..., min_length=1, max_length=5)
    quantity: int = Field(..., gt=0)

@app.route('/orders', methods=['POST'])
def create_order():    
    req = CreateOrderSchema(**request.json)    
    result = create_order_service(req.ticker, req.quantity)    
    return jsonify({'id': result.id}), 201
```

- a) The client sends a JSON body of {"ticker": "", "quantity": -3}. What happens at runtime, and what HTTP response will the client see if no global error handler is registered?
- b) What single addition to the Flask app would cause this to return a clean 400 JSON error response with field-level details, instead of an unhandled exception?

## Part III — Short Essays

1. Two sibling components, PortfolioList and HoldingsPanel, both need to know which portfolio is currently selected. Explain why neither component can own this state directly, what "lifting state up" means in this context, and how data flows in both directions through the parent Dashboard component once the state has been lifted.

2. Explain the rule by which the browser chooses between two CSS rules that target the same element. Use the order ID > class > element to anchor your answer, and explain why the CSS handout recommends keeping specificity LOW by preferring classes over IDs for styling.

## Answers

### Part I — Multiple Choice - Answers

1. Answer: c. === checks both VALUE and TYPE. '5' is a string and 5 is a number, so the comparison is false. (If you used == instead, you would get true because == coerces.) The handout's rule: always default to ===.

2. Answer: b. CSS specificity is ID > class > element. Order in the file only breaks ties at equal specificity. An ID always beats a class regardless of order.

3. Answer: c. The dependency array is the trigger list — the effect runs after the first render and any time any listed value differs from the previous render. Empty array would be (a); no array would be (b).

4. Answer: b. Optional[str] = Union[str, None], paired with = None as the default. Option (a) is required (the ellipsis); option (c) is contradictory (Optional but Field(...) makes it required); option (d) is required and disallows None.

### Part II — Code Evaluation - Answers

1. Question 1

- a) ['AAPL', 'MSFT', 'GOOG']
   ['aapl', 'msft', 'goog']  (the original is unchanged)
   false
- b) .map() always returns a NEW array — even if every element happened to be identical, the array reference is different. === on arrays compares references, so even two arrays containing the same items are not === to each other. This is the same reference-equality concept React relies on for state updates.

2. Question 2

- a) Line A mutates the existing array in place; the array's reference is unchanged. Line B then calls setItems with the SAME reference React already has. React's bailout uses Object.is (essentially reference equality) to decide whether to re-render — same reference → bailout → no re-render. The data is updated under the hood, but React has no signal that anything changed.

- b) Replace lines A and B with a single line that produces a NEW array:

```javascript
function addItem(name) {  
    setItems([...items, name]);  // or, equivalently, the functional form:  // setItems(prev => [...prev, name]);}
```

3. Question 3

- a) Pydantic raises a ValidationError because (i) ticker fails the min_length=1 constraint and (ii) quantity fails gt=0. The exception is unhandled, so Flask falls back to its default error handler and returns a generic HTML 500 Internal Server Error response. The service function is never called.

- b) Register a global error handler for ValidationError that converts the exception into a 400 JSON response, e.g.:

```python
from pydantic import ValidationError
@app.errorhandler(ValidationError)
def handle_validation(err):
    return jsonify({'errors': err.errors()}), 400
```

### Part III — Short Essays - Answers

1. Sibling components in React cannot share state directly — they have no parent-child relationship, and props only flow downward, so PortfolioList has no way to send a value to HoldingsPanel and vice versa. The fix is to lift the state to their closest common ancestor, the Dashboard. Dashboard declares useState(null) for selectedPortfolioId and passes the value down as a prop to both children: PortfolioList reads it to highlight the active card, and HoldingsPanel reads it to choose which holdings to display. To let PortfolioList trigger a change, Dashboard also passes a callback prop named onSelect. When the user clicks a card, PortfolioList calls onSelect(portfolio.id), which actually executes Dashboard's handler, which calls setSelectedPortfolioId. State updates in the parent, both children re-render with the new value, and the UI stays consistent. This is the cardinal pattern of the Dashboard in the kiwi app: data flows DOWN through props, events flow UP through callback props, and shared state lives in the common ancestor.

2. When multiple rules target the same element, the browser uses CSS specificity to decide which one wins. The hierarchy is ID > class > element selector: an ID outranks any combination of class and element selectors; a class outranks any combination of element selectors; and so on. When two rules have equal specificity, the one declared LATER in the stylesheet wins — order is only the tiebreaker. The handout recommends keeping specificity low (relying on classes rather than IDs for styling) because high-specificity rules are hard to override. Once a rule uses an ID, the only way to override it cleanly is to write an even more specific rule, which means more IDs or !important — an escalation arms race that makes a stylesheet brittle and painful to maintain. Reserving IDs for JavaScript hooks and using classes for styling lets developers compose and override styles using ordinary cascade rules.