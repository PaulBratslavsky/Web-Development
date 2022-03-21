# Simple Form Example

```javascript
import React, { useState } from "react";

function TextInput({
  name,
  type,
  placeholder,
  value,
  onChange,
  required,
  label,
  ...rest
}) {
  return (
    <div>
      {label && <label htmlFor={name}>{label}</label>}

      <input
        name={name}
        type={type}
        placeholder={placeholder}
        value={value}
        onChange={onChange}
        required={required}
        {...rest}
      />
    </div>
  );
}

const INITIAL_FORM_STATE = {
  identifier: "",
  password: "",
};

export default function RandomForm() {
  const [formData, setFormData] = useState(INITIAL_FORM_STATE);

  function handleChange(e) {
    const { name, value } = e.target;
    setFormData((prevState) => ({ ...prevState, [name]: value }));
  }

  async function handleSubmit(e) {
    e.preventDefault();

    const data = {
      identifier: formData.identifier,
      password: formData.password,
    };

    const response = await fetch("http://localhost:1337/api/auth/local", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });

    const responseData = await response.json();

    console.log(responseData, "DID NOT WORK");

    alert(JSON.stringify(responseData));
  }

  return (
    <div>
      <h1>Random Form</h1>

      <form onSubmit={handleSubmit}>
        <TextInput
          name="identifier"
          type="email"
          placeholder="Email"
          value={formData.identifier}
          onChange={handleChange}
          required
        />

        <TextInput
          label={"THIS IS YOU EMAIL"}
          name="password"
          type="password"
          placeholder="Password"
          value={formData.password}
          onChange={handleChange}
          required
        />

        <button type={"submit"}>Submit</button>
      </form>
    </div>
  );
}
```
