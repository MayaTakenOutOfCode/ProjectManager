<script lang="ts">
    import { onMount } from "svelte";
  
    // Interface for project data
    interface Project {
      id: number;
      name: string;
      startDate: string;
      deadline: string;
      deadlineHour: string;
      todos: string[];
      dones: string[];
    }
  
    let projects: Project[] = [];
    let projectName = "";
    let projectStartDate = "";
    let projectDeadline = "";
    let projectDeadlineHour = "";
    let newTodo = "";
    let editingProject: Project | null = null;
  
    // Load projects from cookies on mount
    onMount(() => {
      const savedProjects = getProjectsFromCookies();
      if (savedProjects) {
        projects = savedProjects;
      }
    });
  
    // Save projects to cookies
    const saveProjectsToCookies = () => {
      document.cookie = `projects=${JSON.stringify(projects)}; path=/; max-age=31536000`; // Store for 1 year
    };
  
    // Retrieve projects from cookies
    const getProjectsFromCookies = (): Project[] | null => {
      const cookies = document.cookie.split("; ");
      const projectCookie = cookies.find((cookie) => cookie.startsWith("projects="));
      if (projectCookie) {
        const jsonData = projectCookie.split("=")[1];
        return JSON.parse(jsonData) as Project[];
      }
      return null;
    };
  
    // Handle form submission to add or edit a project
    const handleSubmit = (event: Event) => {
      event.preventDefault();
      if (editingProject) {
        // Update an existing project
        editingProject.name = projectName;
        editingProject.startDate = projectStartDate;
        editingProject.deadline = projectDeadline;
        editingProject.deadlineHour = projectDeadlineHour;
        editingProject = null;
      } else {
        // Add a new project
        projects = [
          ...projects,
          {
            id: Date.now(),
            name: projectName,
            startDate: projectStartDate,
            deadline: projectDeadline,
            deadlineHour: projectDeadlineHour,
            todos: [],
            dones: [],
          },
        ];
      }
      saveProjectsToCookies(); // Save projects to cookies
      clearFields();
    };
  
    // Clear the input fields
    const clearFields = () => {
      projectName = "";
      projectStartDate = "";
      projectDeadline = "";
      projectDeadlineHour = "";
    };
  
    // Add a to-do to a project
    const addTodo = (project: Project) => {
      if (newTodo.trim()) {
        project.todos.push(newTodo);
        newTodo = "";
        saveProjectsToCookies(); // Save updated project to cookies
      }
    };
  
    // Move a to-do to done
    const markAsDone = (project: Project, todo: string) => {
      project.todos = project.todos.filter((item) => item !== todo);
      project.dones.push(todo);
      saveProjectsToCookies(); // Save updated project to cookies
    };
  
    // Edit a project
    const editProject = (project: Project) => {
      projectName = project.name;
      projectStartDate = project.startDate;
      projectDeadline = project.deadline;
      projectDeadlineHour = project.deadlineHour;
      editingProject = project;
    };
  
    // Delete a project
    const deleteProject = (id: number) => {
      projects = projects.filter((project) => project.id !== id);
      saveProjectsToCookies(); // Save updated project list to cookies
    };
  </script>
  
  <h1>Project Tracker</h1>
  
  <form on:submit={handleSubmit}>
    <label for="projectName">Project Name:</label>
    <input
      id="projectName"
      type="text"
      placeholder="Enter project name"
      bind:value={projectName}
    />
  
    <label for="projectStartDate">Start Date:</label>
    <input id="projectStartDate" type="date" bind:value={projectStartDate} />
  
    <label for="projectDeadline">Deadline Date:</label>
    <input id="projectDeadline" type="date" bind:value={projectDeadline} />
  
    <label for="projectDeadlineHour">Deadline Time:</label>
    <input
      id="projectDeadlineHour"
      type="time"
      bind:value={projectDeadlineHour}
    />
  
    <input
      type="submit"
      value={editingProject ? "Update Project" : "Add Project"}
    />
  </form>
  
  <div id="project-list">
    {#each projects as project}
      <div class="project-card">
        <div class="project-details">
          <span><strong>Name:</strong> {project.name}</span>
          <span><strong>Start Date:</strong> {project.startDate}</span>
          <span
            ><strong>Deadline:</strong>
            {project.deadline} at {project.deadlineHour}</span
          >
        </div>
        <div class="todo-input">
          <input type="text" placeholder="Add a to-do" bind:value={newTodo} />
          <button on:click={() => addTodo(project)}>Add</button>
        </div>
        <ul class="todo-list">
          {#each project.todos as todo}
            <li>
              {todo}
              <button class="mark-done" on:click={() => markAsDone(project, todo)}
                >Done</button
              >
            </li>
          {/each}
        </ul>
        <ul class="done-list">
        <h3 style="color:palevioletred; font-size:160%;">Done</h3>
          {#each project.dones as done}
            <li>{done}</li>
          {/each}
        </ul>
        <button class="edit" on:click={() => editProject(project)}>Edit</button>
        <button class="delete" on:click={() => deleteProject(project.id)}>Delete</button>
      </div>
    {/each}
  </div>
  
  <style>
    :global(body) {
      background-color: #fff0f6;
      font-family: "Poppins", sans-serif;
      margin: 0;
      padding: 0;
      text-align: center;
    }
  
    h1 {
      font-size: 2.5rem;
      color: #d63384;
    }
  
    form {
      background: #ffe6f3;
      border-radius: 15px;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
      padding: 20px;
      max-width: 500px;
      margin: 20px auto;
    }
  
    label {
      font-size: 1.1rem;
      color: #d63384;
      font-weight: 600;
      display: block;
      margin-bottom: 5px;
    }
  
    input[type="date"],
    input[type="time"],
    input[type="text"],
    input[type="submit"] {
      width: 100%;
      padding: 10px;
      margin-bottom: 15px;
      border: 1px solid #f5c2e0;
      border-radius: 10px;
      font-size: 1rem;
      background-color: #fff5f8;
    }
  
    input[type="submit"] {
      background-color: #d63384;
      color: white;
      font-weight: bold;
      cursor: pointer;
      transition: background-color 0.3s;
    }
  
    input[type="submit"]:hover {
      background-color: #ad1c61;
    }
  
    #project-list {
      max-width: 800px;
      margin: 20px auto;
      display: inline-flex;
    }
  
    .project-card {
      background-color: #fff5f8;
      padding: 20px;
      margin-bottom: 10px;
      border: 1px solid #f5c2e0;
      border-radius: 15px;
      text-align: left;
    }
  
    .project-details span {
      display: block;
      margin-bottom: 5px;
    }
  
    .todo-input {
      display: flex;
      gap: 10px;
      margin-top: 10px;
    }
  
    .todo-input input {
      flex: 1;
      padding: 8px;
      border: 1px solid #f5c2e0;
      border-radius: 10px;
      background-color: #fff5f8;
    }
  
    .todo-input button {
      padding: 8px 12px;
      background-color: #d63384;
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      transition: background-color 0.3s;
    }
  
    .todo-input button:hover {
      background-color: #ad1c61;
    }
  
    .todo-list,
    .done-list {
      margin-top: 10px;
    }
  
    .todo-list li,
    .done-list li {
      margin-bottom: 5px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
  
    .done-list {
      color: #6c757d;
    }
  
    .mark-done {
      background-color: #51cf66;
      color: white;
      border: none;
      border-radius: 5px;
      padding: 5px 10px;
      cursor: pointer;
      transition: background-color 0.3s;
    }
  
    .mark-done:hover {
      background-color: #37b24d;
    }
    button {
      border: none;
      border-radius: 10px;
      padding: 10px 15px;
      font-size: 1rem;
      font-weight: bold;
      cursor: pointer;
      transition:
        background-color 0.3s ease,
        transform 0.2s ease;
    }
  
    button:hover {
      transform: scale(1.05);
    }
  
    /* Style for Add Button */
    .todo-input button {
      background-color: #d63384;
      color: white;
    }
  
    .todo-input button:hover {
      background-color: #ad1c61;
    }
  
    /* Style for Edit Button */
    button.edit {
      background-color: #f06595;
      color: white;
    }
  
    button.edit:hover {
      background-color: #e64980;
    }
  
    /* Style for Delete Button */
    button.delete {
      background-color: #fa5252;
      color: white;
    }
  
    button.delete:hover {
      background-color: #d90429;
    }
  </style>
  