# Post Data Strapi 4

## With Image Upload

```javascript
async function handleSubmit(event) {
  event.preventDefault();
  const formData = new FormData();

  if (description && file) {
    formData.append("data", JSON.stringify({ description }));
    formData.append("files.image", file);

    try {
      const response = await fetch(baseURL + filter, {
        method: "POST",
        headers: {
          Authorization: `Bearer ${user.jwt}`,
        },
        body: formData,
      });

      const data = await response.json();
      console.log(data, "RESPONSE");

      setDescription("");
      imageInputRef.current.value = "";
      setFile(null);
      alert("Form Submitted");

      history.push("/");
    } catch (error) {
      console.error("EXCEPTION: ", error);
    }
  } else {
    alert("Please add description and (or) file");
  }
}
```

```javascript
import { useState, useContext } from "react";
import Table from "../components/Table/Table";
import Modal from "../components/Modal/Modal";
import CreateTeamForm from "../components/CreateTeamForm/CreateTeamForm";
import { UsersIcon } from "@heroicons/react/outline";
import ActionHeader from "../components/ActionHeader/ActionHeader";
import TableColumn from "../components/Table/TableColumn";
import useFetchQuery from "../hooks/useFetchQuery";
import { baseUrl } from "../config";
const teamsUrl = `${baseUrl}/api/teams`;

export default function Teams() {
  const [open, setOpen] = useState(false);
  const { data, loading, error } = useFetchQuery(teamsUrl);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error :(</p>;

  const teams = data ? data.data.attributes.results : [];

  return (
    <div>
      <ActionHeader
        title="TEAMS"
        count={teams.length}
        cta="Create Team"
        ctaAction={() => setOpen(true)}
      />

      <div className="h-full overflow-x-auto">
        <Table sourceData={teams}>
          <TableColumn source="teamName" label="Team Name" />
          <TableColumn source="teamDescription" label="Description" />
          <TableColumn
            source="teamOwner"
            label="Founder"
            render={(data) =>
              data?.firstName ? `${data.firstName} ${data.lastName}` : "N/A"
            }
          />
          <TableColumn
            source="teamMembers"
            label="Members"
            render={(data) => data.length}
          />
        </Table>
      </div>

      <Modal
        title="Creat a team"
        iconComponent={UsersIcon}
        description="Create a team to organize your projects and collaborate."
        open={open}
        setOpen={setOpen}
      >
        <CreateTeamForm />
      </Modal>
    </div>
  );
}
```

## example with auth

```javascript
async function handleSubmit(event) {
  event.preventDefault();

  const hasErrors = formValidation(formData);
  const teamUrl = "http://localhost:1337/api/teams";

  if (!hasErrors) {
    try {
      const response = await fetch(teamUrl, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },

        body: JSON.stringify({ data: formData }),
      });

      const data = await response.json();
      console.log(data);
      
    } catch (error) {
      console.log(error);
      
    } finally {
      setOpen(false);
    }
  }
}
```
