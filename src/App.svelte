<script>
  import { setContext, onMount, afterUpdate } from "svelte";
  // components
  import Navbar from "./Navbar.svelte";
  import ExpensesList from "./ExpensesList.svelte";
  import Totals from "./Totals.svelte";
  import ExpenseForm from "./ExpenseForm.svelte";
  import Modal from "./Modal.svelte";

  let expenses = [];
  let appName = "";
  let appAmount = null;
  let isFormOpen = false;
  let isEditing=false;
  let id=null;

  //local storage
  onMount(() => {
    expenses = localStorage.getItem("expenses")
      ? JSON.parse(localStorage.getItem("expenses"))
      : [];
  });

  afterUpdate(() => {
     localStorage.setItem("expenses", JSON.stringify(expenses));
  });
 

  $: total = expenses.reduce((acc, curr) => {
    return (acc += curr.amount);
  }, 0);

 
  function hideForm() {
    isFormOpen = false;
    appName = "";
    appAmount = null;
    id = null;
  }


  // context
  setContext("remove", removeExpense);
  setContext("modify", setModifiedExpense);

  function showForm() {
    isFormOpen = true;
  }

 function clearExpenses() {
    expenses = [];
  }
  function addExpense({ name, amount }) {
    let expense = { id: Math.random() * Date.now(), name, amount };
    expenses = [expense, ...expenses];
  }

 
  function setModifiedExpense(id) {
    let expense = expenses.find(item => item.id === id);
    isEditing=true;
    id=id;
    appName = expense.name;
    appAmount = expense.amount;
    showForm();
  }

 function editExpense({ name, amount }) {
    expenses = expenses.map(item => {
      return item.id === id ? { name, amount } : item;
    });

 function removeExpense(id) {
    expenses = expenses.filter(item => item.id !== id);
  }

</script>

<Navbar {showForm} />
<main class="content">
  {#if isFormOpen}
    <Modal>
      <ExpenseForm
        {addExpense}
        name={appName}
        amount={appAmount}
        {isEditing}
        {editExpense}
        {hideForm} />
    </Modal>
  {/if}
  <Totals title="total expenses" {total} />
  <ExpensesList {expenses} />
  <button
    type="button"
    class="btn btn-primary btn-block"
    on:click={clearExpenses}>
    clear expenses
  </button>
</main>
