# React and CRUD APIs

## Starter

```
npm i

npm i json-server

node server.js

http://localhost:3001/pets/

npm run start

```

## Fetch a List of Data with useEffect and Promises

Use the useEffect hook to kick off the data fetch to our json-server back end, then use Promises to handle the response.

- import the useEffect hook from React

- the function is going to run after our component renders

- calling set pets is going to trigger a rerender, and we should see our pets

`{JSON.stringify(pets, null, 2)}`

- initialize our state to an empty array

```js
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";
import Pet from "./Pet";
import "./index.css";

const App = () => {
  const [pets, setPets] = useState([]);

  useEffect(() => {
    fetch("http://localhost:3001/pets")
      .then((res) => res.json())
      .then((pets) => setPets(pets));
  }, []);

  return (
    <main>
      <h1>Adopt-a-Pet</h1>
      <ul>
        {pets.map((pet) => (
          <li key={pet.id}>
            <Pet pet={pet} />
          </li>
        ))}
      </ul>
      <button>Add a Pet</button>
    </main>
  );
};

ReactDOM.render(<App />, document.querySelector("#root"));
```

Asnyc await

If you want to do async/await inside a useEffect, you need to write an async function and put that inside the effect function, instead of making the effect function itself async or as we saw before, you can always just use promise chaining.

- create a function inside the useEffect mark it as async
- fetch returns a promise, we can await that promise
- causes the code to stop at this line until the fetch returns
- create a variable called pets and await res.json
- can call setPets with the array of pets

```js
useEffect(() => {
  async function getData() {
    const res = await fetch(
      'http://localhost:3001/pets'
    );
    const pets = await res.json();
    setPets(pets);
  }
  getData();
```

## Display a Loading Indicator

- add a loading indicator so that when the pets are loading, we don't just see nothing
- create a new piece of state called is loading initialized the value of false
- set loading to true before we start fetching data, and set loading to false after we're done
- one problem with how we're setting our loading indicator back to false is that if a fetch or the JSON parsing fails, it's going to throw an error. Loading will never be set to false.
- wrap it in a try catch. We can wrap all this in a try block and if an error occurs, we'll catch the error and then we can set loading to false in the catch block
- to see how this might work for promises, we can comment out our async-await version and comment back in our promise version

```js
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";
import Pet from "./Pet";
import "./index.css";

const App = () => {
  const [pets, setPets] = useState([]);
  const [isLoading, setLoading] = useState(false);

  useEffect(() => {
    // async function getData() {
    //   setLoading(true);
    //   try {
    //     const res = await fetch(
    //       'http://localhost:3001/pets'
    //     );
    //     const pets = await res.json();
    //     setPets(pets);
    //     setLoading(false);
    //   } catch (e) {
    //     setLoading(false);
    //   }
    // }
    // getData();

    setLoading(true);
    fetch("http://localhost:3001/pets")
      .then((res) => res.json())
      .then((pets) => setPets(pets))
      .finally(() => setLoading(false));
  }, []);

  return (
    <main>
      <h1>Adopt-a-Pet</h1>
      {isLoading ? (
        <div className="loading">Loading...</div>
      ) : (
        <>
          <ul>
            {pets.map((pet) => (
              <li key={pet.id}>
                <Pet pet={pet} />
              </li>
            ))}
          </ul>
          <button>Add a Pet</button>
        </>
      )}
    </main>
  );
};

ReactDOM.render(<App />, document.querySelector("#root"));
```

```js

```

## Create a Pets Component

```js
import React from "react";

const Pet = ({ pet, onEdit, onRemove }) => {
  return (
    <div className="pet">
      {pet.photo ? (
        <img src={pet.photo} alt="" className="pet-photo sm" />
      ) : (
        <div className="no-photo">?</div>
      )}
      <button className="pet-name" onClick={onEdit}>
        {pet.name}
      </button>
      <div className="pet-kind">{pet.kind}</div>

      <button className="adopt-btn" onClick={onRemove}>
        <span role="img" aria-label="adopt this pet">
          🏠
        </span>
      </button>
    </div>
  );
};

export default Pet;
```

## Display a Modal Dialog Using react-modal

When we click this add a pet button, we'd like to pop open a modal dialog with a form that will let the user type in the name and upload a photo for the pet.

```
npm i react-modal
```

- import it as modal in index.js `import Modal from 'react-modal';`
- add it to the bottom of main

```js
<main>
  ...
  <Modal isOpen={isNewPetOpen} onRequestClose={() => setNewPetOpen(false)}>
    hello
  </Modal>
</main>
```

- the modal requires a prop called is open to tell it whether to display or not

```js
const [isNewPetOpen, setNewPetOpen] = useState(false);
```

- when we click the add a pet button, we want to turn that state true. onClick for this button will pass a function and it's going to call setNewPetOpen to true. Now when we click the button, modal shows up, but we have no way out

```js
<>
  <ul>
    {pets.map((pet) => (
      <li key={pet.id}>
        <Pet pet={pet} />
      </li>
    ))}
  </ul>
  <button onClick={() => setNewPetOpen(true)}>Add a Pet</button>
</>
```

- pass another prop to the modal called onRequestClose and we'll pass in a function that will call setNewPetOpen to false.

```js
<Modal isOpen={isNewPetOpen} onRequestClose={() => setNewPetOpen(false)}>
  hello
</Modal>
```

- note the error

```js
const el = document.querySelector("#root");
Modal.setAppElement(el);
ReactDOM.render(<App />, el);
```

## Create a New Pet Form

Inside the modal dialog, we need a form to input the pet's data.

NewPetModal.js:

```js
import React, { useState } from "react";
import Modal from "react-modal";

const NewPetModal = ({ onCancel }) => {
  const [name, setName] = useState("");
  const [kind, setKind] = useState("");

  return (
    <Modal isOpen={true} onRequestClose={onCancel}>
      <h2>New Pet</h2>
      <form className="pet-form">
        <label htmlFor="name">Name</label>
        <input
          type="text"
          id="name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
        <label htmlFor="kind">Kind</label>
        <select
          name="kind"
          id="kind"
          value={kind}
          onChange={(e) => setKind(e.target.value)}
        >
          <option value="">Choose a kind</option>
          <option value="cat">Cat</option>
          <option value="dog">Dog</option>
        </select>
        <button type="button" onClick={onCancel}>
          Cancel
        </button>
        <button type="submit">Save</button>
      </form>
    </Modal>
  );
};

export default NewPetModal;
```

In index.js

```js
import NewPetModal from './NewPetModal';

...
<main>
...
  {isNewPetOpen && (
    <NewPetModal
      isOpen={isNewPetOpen}
      onCancel={() => setNewPetOpen(false)}
    />
  )}
</main>
```

## Add a Photo

```js
import React, { useState, useRef } from "react";
import Modal from "react-modal";

const NewPetModal = ({ onCancel }) => {
  const [name, setName] = useState("");
  const [kind, setKind] = useState("");
  const [photo, setPhoto] = useState(null);
  const photoInput = useRef();

  const updatePhoto = () => {
    const file = photoInput.current.files && photoInput.current.files[0];

    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => setPhoto(reader.result);
      reader.readAsDataURL(file);
    }
  };

  return (
    <Modal isOpen={true} onRequestClose={onCancel}>
      <h2>New Pet</h2>
      <form className="pet-form">
        {photo && <img alt="the pet" src={photo} />}
        <label htmlFor="photo">Photo</label>
        <input type="file" id="photo" ref={photoInput} onChange={updatePhoto} />
        <label htmlFor="name">Name</label>
        <input
          type="text"
          id="name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
        <label htmlFor="kind">Kind</label>
        <select
          name="kind"
          id="kind"
          value={kind}
          onChange={(e) => setKind(e.target.value)}
        >
          <option value="">Choose a kind</option>
          <option value="cat">Cat</option>
          <option value="dog">Dog</option>
        </select>
        <button type="button" onClick={onCancel}>
          Cancel
        </button>
        <button type="submit">Save</button>
      </form>
    </Modal>
  );
};

export default NewPetModal;
```

## Implement Saving Pet Data Locally

The New Pet form currently doesn't do anything when you click Save, so in this lesson we'll remedy that by adding a pet to our local list of pets. This will lay the groundwork for making the call to the server.

- add an on submit handler to our form to intercept that event

```js
import React, { useState, useRef } from "react";
import Modal from "react-modal";

const NewPetModal = ({ onCancel, onSave }) => {
  const [name, setName] = useState("");
  const [kind, setKind] = useState("");
  const [photo, setPhoto] = useState(null);
  const photoInput = useRef();

  const updatePhoto = () => {
    const file = photoInput.current.files && photoInput.current.files[0];

    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => setPhoto(reader.result);
      reader.readAsDataURL(file);
    }
  };

  const submit = (event) => {
    event.preventDefault();
    onSave({
      name,
      kind,
      photo,
    });
  };
  return (
    <Modal isOpen={true} onRequestClose={onCancel}>
      <h2>New Pet</h2>
      <form className="pet-form" onSubmit={submit}>
        {photo && <img alt="the pet" src={photo} />}
        <label htmlFor="photo">Photo</label>
        <input type="file" id="photo" ref={photoInput} onChange={updatePhoto} />
        <label htmlFor="name">Name</label>
        <input
          type="text"
          id="name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
        <label htmlFor="kind">Kind</label>
        <select
          name="kind"
          id="kind"
          value={kind}
          onChange={(e) => setKind(e.target.value)}
        >
          <option value="">Choose a kind</option>
          <option value="cat">Cat</option>
          <option value="dog">Dog</option>
        </select>
        <button type="button" onClick={onCancel}>
          Cancel
        </button>
        <button type="submit">Save</button>
      </form>
    </Modal>
  );
};

export default NewPetModal;
```

- create addPet in index.js

```js
const addPet = async ({ name, kind, photo }) => {
  setPets([
    ...pets,
    {
      id: Math.random(),
      name,
      kind,
      photo,
    },
  ]);
  setNewPetOpen(false);
};
```

- pass it as onSave into the modal from index.js

```js
  {isNewPetOpen && (
    <NewPetModal
      onCancel={() => setNewPetOpen(false)}
      onSave={addPet}
    />
  )}
</main>
```

- this save to local state only, refreshing will cause the new pet to disappear

## Use HTTP POST to Save the Pet to the Server

The app is currently saving the pet locally, but it's not persisted to the server.

We'll add the HTTP POST call to save the pet data, and refresh the list. We'll also implement a saving state, disable the buttons while the save is underway, and display any errors that the server returns.

- modify the addPet function to actually persist this to the server in a new api.js file

```js
const handleErrors = (res) => {
  if (!res.ok) {
    return res.json().then((error) => {
      throw error;
    });
  }
  return res;
};

export const listPets = () => {
  return fetch("http://localhost:3001/pets").then((res) => res.json());
};

export const createPet = (pet) => {
  return fetch("http://localhost:3001/pets", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(pet),
  })
    .then(handleErrors)
    .then((res) => res.json());
};
```

index.js:

```js
import { listPets, createPet } from './api';

...

  useEffect(() => {
    setLoading(true);
    listPets()
      .then(pets => setPets(pets))
      .finally(() => setLoading(false));
  }, []);

  const addPet = async pet => {
    return createPet(pet).then(newPet => {
      setPets([...pets, newPet]);
      setNewPetOpen(false);
    });
  };
```

Possible server error - cannot find module encodings.

- Stop the server and npm install json-server
- restart the server
- test with a blank pet to see the server errors

Catch and display the rrors in NewPetModal.js

```js
  const [errors, setErrors] = useState(null);
  ...
const submit = (event) => {
  event.preventDefault();
  setSaving(true);
  onSave({
    name,
    kind,
    photo,
  }).catch((error) => {
    console.log(error);
    setErrors(error);
    setSaving(false);
  });
};
```

Add errors display to the form:

```js
    <Modal isOpen={true} onRequestClose={onCancel}>
      <h2>New Pet</h2>
      <form className="pet-form" onSubmit={submit}>
        {photo && <img alt="the pet" src={photo} />}
        <label htmlFor="photo">Photo</label>
        <input
          type="file"
          id="photo"
          ref={photoInput}
          onChange={updatePhoto}
        />
        <label htmlFor="name">Name</label>
        <input
          type="text"
          id="name"
          value={name}
          onChange={e => setName(e.target.value)}
        />
        {errors && errors.name && (
          <div className="error">{errors.name}</div>
        )}
        <label htmlFor="kind">Kind</label>
        <select
          name="kind"
          id="kind"
          value={kind}
          onChange={e => setKind(e.target.value)}
        >
          <option value="">Choose a kind</option>
          <option value="cat">Cat</option>
          <option value="dog">Dog</option>
        </select>
        {errors && errors.kind && (
          <div className="error">{errors.kind}</div>
        )}
        <button
          disabled={saving}
          type="button"
          onClick={onCancel}
        >
          Cancel
        </button>
        <button disabled={saving} type="submit">
          Save
        </button>
      </form>
    </Modal>
  );
```

disable the form while its being submitted

```js
const [saving, setSaving] = useState(false);
...
const submit = event => {
    event.preventDefault();
    setSaving(true);
    onSave({
      name,
      kind,
      photo
    }).catch(error => {
      console.log(error);
      setErrors(error);
      setSaving(false);
    });
  };
```

disable the buttons while saving is true

```js
<button
  disabled={saving}
  type="button"
  onClick={onCancel}
>
  Cancel
</button>
<button disabled={saving} type="submit">
  Save
</button>
```

Test with valid inputs

## Use HTTP PUT to Update the Pet on the Server

We need to be able to click on a pet to edit its details. We'll create a new component for an Edit modal and its form.

We're going to need:

- a dialogue for being able to edit the pet
- some state to control whether that dialogue is open or not
- a new API call to be able to save that pet to the server

New state in App

```js
const [currentPet, setCurrentPet] = useState(null);
```

Pass onEdit to the pet component:

```js
Pet
  pet={pet}
  onEdit={() => {
    console.log('set', pet);
    setCurrentPet(pet);
  }}
/>
```

Import editPetModal in index.js

`import EditPetModal from './EditPetModal';`

A new form editPetModal.js. The form is going to be pretty much the same, but there's a couple differences. We want to say editPet instead of newPet and we want to initialize this state to whatever pet is passed in, whatever that currentPet is instead of empty.

We'll accept a pet prop `const EditPetModal = ({ pet, onCancel, onSave }) =>` and initialize the name photo and kind in state:

```js
const [name, setName] = useState(pet.name);
const [kind, setKind] = useState(pet.kind);
const [photo, setPhoto] = useState(pet.photo);
```

```js
import React, { useState, useRef } from "react";
import Modal from "react-modal";

const EditPetModal = ({ pet, onCancel, onSave }) => {
  const [name, setName] = useState(pet.name);
  const [kind, setKind] = useState(pet.kind);
  const [photo, setPhoto] = useState(pet.photo);
  const [errors, setErrors] = useState(null);
  const [saving, setSaving] = useState(false);
  const photoInput = useRef();

  const updatePhoto = () => {
    const file = photoInput.current.files && photoInput.current.files[0];

    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => setPhoto(reader.result);
      reader.readAsDataURL(file);
    }
  };

  const submit = (event) => {
    event.preventDefault();
    setSaving(true);
    onSave({
      ...pet, // id!
      name,
      kind,
      photo,
    }).catch((error) => {
      console.log(error);
      setErrors(error);
      setSaving(false);
    });
  };
  return (
    <Modal isOpen={true} onRequestClose={onCancel}>
      <h2>Edit Pet</h2>
      <form className="pet-form" onSubmit={submit}>
        {photo && <img alt="the pet" src={photo} />}
        <label htmlFor="photo">Photo</label>
        <input type="file" id="photo" ref={photoInput} onChange={updatePhoto} />
        <label htmlFor="name">Name</label>
        <input
          type="text"
          id="name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
        {errors && errors.name && <div className="error">{errors.name}</div>}
        <label htmlFor="kind">Kind</label>
        <select
          name="kind"
          id="kind"
          value={kind}
          onChange={(e) => setKind(e.target.value)}
        >
          <option value="">Choose a kind</option>
          <option value="cat">Cat</option>
          <option value="dog">Dog</option>
        </select>
        {errors && errors.kind && <div className="error">{errors.kind}</div>}
        <button disabled={saving} type="button" onClick={onCancel}>
          Cancel
        </button>
        <button disabled={saving} type="submit">
          Save
        </button>
      </form>
    </Modal>
  );
};

export default EditPetModal;
```

Go back to our index file and render the editPet modal down at the bottom

```js
  {isNewPetOpen && (
    <NewPetModal
      onCancel={() => setNewPetOpen(false)}
      onSave={addPet}
    />
  )}
  {currentPet && (
    <EditPetModal
      pet={currentPet}
      onCancel={() => setCurrentPet(null)}
      onSave={savePet}
    />
  )}
</main>
```

A savePet function in index.js:

```js
const savePet = async (pet) => {
  // api call goes here
};
```

Test by clicking on a pet.

We'll need an update function in api:

`import { listPets, createPet, updatePet } from './api';`

Then in api.js:

```js
export const updatePet = (pet) => {
  console.log(pet);
  return fetch(`http://localhost:3001/pets/${pet.id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(pet),
  })
    .then(handleErrors)
    .then((res) => res.json());
};
```

Back in index:

```js
const savePet = async (pet) => {
  return updatePet(pet).then((updatedPet) => {
    setPets((pets) =>
      pets.map((pet) => (pet.id === updatedPet.id ? updatedPet : pet))
    );
    setCurrentPet(null);
  });
};
```

## Refactor New and Edit Forms into One

The "New Pet" and "Edit Pet" forms are very similar, so in this lesson we'll look at how to refactor them into one form component and DRY up the code.

The only real difference between our two forms is the heading and how the state is initialized.

- make a file called PetForm.js, import React and create a component
- copy the form only from edit pet modal

```js
import React from 'react';

const PetForm = () => {
return (
    <form className="pet-form" onSubmit={submit}>
      {photo && <img alt="the pet" src={photo} />}
      <label htmlFor="photo">Photo</label>
      <input
        type="file"
        id="photo"
        ref={photoInput}
        onChange={updatePhoto}
      />
      <label htmlFor="name">Name</label>
      <input
        type="text"
        id="name"
        value={name}
        onChange={e => setName(e.target.value)}
      />
      {errors && errors.name && (
        <div className="error">{errors.name}</div>
      )}
      <label htmlFor="kind">Kind</label>
      <select
        name="kind"
        id="kind"
        value={kind}
        onChange={e => setKind(e.target.value)}
      >
        <option value="">Choose a kind</option>
        <option value="cat">Cat</option>
        <option value="dog">Dog</option>
      </select>
      {errors && errors.kind && (
        <div className="error">{errors.kind}</div>
      )}
      <button
        disabled={saving}
        type="button"
        onClick={onCancel}
      >
        Cancel
      </button>
      <button disabled={saving} type="submit">
        Save
      </button>
    </form>
    ...
```

Add functions and props

```js
import React, { useState, useRef } from "react";

const PetForm = ({ pet, onSave, onCancel }) => {
  const initialPet = pet || {
    name: "",
    kind: "",
    photo: null,
  };
  const [name, setName] = useState(initialPet.name);
  const [kind, setKind] = useState(initialPet.kind);
  const [photo, setPhoto] = useState(initialPet.photo);
  const [errors, setErrors] = useState(null);
  const [saving, setSaving] = useState(false);
  const photoInput = useRef();

  const updatePhoto = () => {
    const file = photoInput.current.files && photoInput.current.files[0];

    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => setPhoto(reader.result);
      reader.readAsDataURL(file);
    }
  };

  const submit = (event) => {
    event.preventDefault();
    setSaving(true);
    onSave({
      ...pet,
      name,
      kind,
      photo,
    }).catch((error) => {
      console.log(error);
      setErrors(error);
      setSaving(false);
    });
  };

  return (
    <form className="pet-form" onSubmit={submit}>
      {photo && <img alt="the pet" src={photo} />}
      <label htmlFor="photo">Photo</label>
      <input type="file" id="photo" ref={photoInput} onChange={updatePhoto} />
      <label htmlFor="name">Name</label>
      <input
        type="text"
        id="name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      {errors && errors.name && <div className="error">{errors.name}</div>}
      <label htmlFor="kind">Kind</label>
      <select
        name="kind"
        id="kind"
        value={kind}
        onChange={(e) => setKind(e.target.value)}
      >
        <option value="">Choose a kind</option>
        <option value="cat">Cat</option>
        <option value="dog">Dog</option>
      </select>
      {errors && errors.kind && <div className="error">{errors.kind}</div>}
      <button disabled={saving} type="button" onClick={onCancel}>
        Cancel
      </button>
      <button disabled={saving} type="submit">
        Save
      </button>
    </form>
  );
};

export default PetForm;
```

EditPetModal.js:

```js
import React from "react";
import Modal from "react-modal";
import PetForm from "./PetForm";

const EditPetModal = ({ pet, onCancel, onSave }) => {
  return (
    <Modal isOpen={true} onRequestClose={onCancel}>
      <h2>Edit Pet</h2>
      <PetForm pet={pet} onCancel={onCancel} onSave={onSave} />
    </Modal>
  );
};

export default EditPetModal;
```

Test editing.

NewPetModal.js

```js
import React from "react";
import Modal from "react-modal";
import PetForm from "./PetForm";

const NewPetModal = ({ onCancel, onSave }) => {
  return (
    <Modal isOpen={true} onRequestClose={onCancel}>
      <h2>New Pet</h2>
      <PetForm onCancel={onCancel} onSave={onSave} />
    </Modal>
  );
};

export default NewPetModal;
```

Handle the case when pet is undefined in PetForm.

```js
const PetForm = ({ pet, onSave, onCancel }) => {
  const initialPet = pet || {
    name: '',
    kind: '',
    photo: null
  };
  const [name, setName] = useState(initialPet.name);
  const [kind, setKind] = useState(initialPet.kind);
  const [photo, setPhoto] = useState(initialPet.photo);
```

Full NewPet form:

```js
import React, { useState, useRef } from "react";

const PetForm = ({ pet, onSave, onCancel }) => {
  const initialPet = pet || {
    name: "",
    kind: "",
    photo: null,
  };
  const [name, setName] = useState(initialPet.name);
  const [kind, setKind] = useState(initialPet.kind);
  const [photo, setPhoto] = useState(initialPet.photo);
  const [errors, setErrors] = useState(null);
  const [saving, setSaving] = useState(false);
  const photoInput = useRef();

  const updatePhoto = () => {
    const file = photoInput.current.files && photoInput.current.files[0];

    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => setPhoto(reader.result);
      reader.readAsDataURL(file);
    }
  };

  const submit = (event) => {
    event.preventDefault();
    setSaving(true);
    onSave({
      ...pet,
      name,
      kind,
      photo,
    }).catch((error) => {
      console.log(error);
      setErrors(error);
      setSaving(false);
    });
  };

  return (
    <form className="pet-form" onSubmit={submit}>
      {photo && <img alt="the pet" src={photo} />}
      <label htmlFor="photo">Photo</label>
      <input type="file" id="photo" ref={photoInput} onChange={updatePhoto} />
      <label htmlFor="name">Name</label>
      <input
        type="text"
        id="name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      {errors && errors.name && <div className="error">{errors.name}</div>}
      <label htmlFor="kind">Kind</label>
      <select
        name="kind"
        id="kind"
        value={kind}
        onChange={(e) => setKind(e.target.value)}
      >
        <option value="">Choose a kind</option>
        <option value="cat">Cat</option>
        <option value="dog">Dog</option>
      </select>
      {errors && errors.kind && <div className="error">{errors.kind}</div>}
      <button disabled={saving} type="button" onClick={onCancel}>
        Cancel
      </button>
      <button disabled={saving} type="submit">
        Save
      </button>
    </form>
  );
};

export default PetForm;
```

## Use HTTP DELETE to Remove a Pet from the Server

When you click the house button it will remove the pet from the list.

api.js

```js
export const deletePet = (pet) => {
  return fetch(`http://localhost:3001/pets/${pet.id}`, {
    method: "DELETE",
  })
    .then(handleErrors)
    .then((res) => res.json());
};
```

In index file, import this delete pet function.

```js
import { listPets, createPet, updatePet, deletePet } from "./api";
```

wire up a handler to the home button to call our delete pet API.

In Pet component we can see it accepts an on remove prop.

In index, we can pass a prop called onRemove. When that's called, we want to do a little bit of work before we call our delete function.

We'll make a function called remove pet using the browsers confirm dialog to make sure that the user actually wants to remove this pet.

If the result is true we'll call our delete pet API, and pass in the pet

```js
const removePet = (byePet) => {
  const result = window.confirm(
    `Are you sure you want to adopt ${byePet.name}`
  );
  if (result) {
    deletePet(byePet).then(() => {
      setPets((pets) => pets.filter((pet) => pet.id !== byePet.id));
    });
  }
};
```

and pass it to Pet:

```js
<Pet
  pet={pet}
  onRemove={() => removePet(pet)}
  onEdit={() => {
    console.log("set", pet);
    setCurrentPet(pet);
  }}
/>
```

And in Pet:

```js
<button className="adopt-btn" onClick={onRemove}>
  <span role="img" aria-label="adopt this pet">
    🏠
  </span>
</button>
```
