<template>
  <div class="projects side-bar">
    <div class="project-form" v-bind:class="{ open: form.isOpen }">
      <input v-model="form.title" type="text" placeholder="Topic" maxlength="40" autofocus>
      <textarea v-model="form.descriptionText" placeholder="Details (optional, uses markdown)"></textarea>
      <button class="add-button" @click="toggleForm">+ Add your hack</button>
    </div>
    <button v-if="user.hasProject" class="add-button" title="You've added a hack!" disabled>Hack Added!</button>
    <button v-else class="add-button" @click="toggleForm">+ Add your hack</button>
    <div v-for="project in projects"
      @dragstart="drag"
      @click="updateSelectedProject(project.id)"
      v-bind:key="project.id"
      v-bind:data-id="project.id"
      v-bind:class="{ selected: selectedProjectId === project.id || project.editMode }"
      v-bind:draggable="!(selectedProjectId === project.id && project.editMode) && (project.username === user.username || user.isAdmin)"
      v-cloak
      class="project">
      <div class="project-title-bar">
        <a class="project-author" v-bind:href="'https://github.com/' + project.username" draggable="false">@{{project.username}}</a>
        <div class="project-title" v-if="!project.editMode">{{project.title}}</div>
        <input class="project-title-editor" v-else v-model="project.title" type="text" maxlength="40" autofocus>
      </div>
      <img
        v-bind:src="selectedProjectId == project.id ? project.avatar : project.avatar_thumbnail"
        v-bind:title="project.username" v-bind:alt="project.username"
        v-bind:data-dragicon="project.avatar_thumbnail"
        class="project-author-avatar" draggable="false" />
      <div v-if="!project.editMode" class="project-description" v-html="project.descriptionHtml"></div>
      <textarea v-else class="project-description-editor" v-model="project.descriptionText"></textarea>
      <div class="action-button-bar" v-if="(selectedProjectId == project.id || project.editMode) && (project.username === user.username || user.isAdmin)">
        <a v-if="!project.editMode" class="edit action-button" @click="toggleEditMode(project.id)">Edit</a>
        <a v-if="!project.editMode" class="delete action-button" @click="deleteComment(project.id)">Delete</a>
        <a v-if="project.editMode" class="cancel action-button" @click="toggleEditMode(project.id)">Cancel</a>
        <a v-if="project.editMode" class="save action-button" @click="updateProject(project)">Save</a>
      </div>
    </div>
    <div class="details" v-if="state === 'running' && projects.length === 0">
      <p>No hacks yet, you can be the first!</p>
    </div>
    <div class="loader" v-if="state === 'init'">
      <div class="loader1"></div>
      <div class="loader2"></div>
      <div class="loader3"></div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'sidebar',
    props: ['form', 'projects', 'selectedProjectId', 'user', 'state'],
    methods: {
      toggleForm () {
        this.$emit('toggleForm')
      },
      drag (event) {
        this.$emit('drag', event)
      },
      updateSelectedProject (projectId) {
        this.$emit('updateSelectedProject', projectId)
      },
      toggleEditMode (projectId) {
        this.$emit('toggleEditMode', projectId)
      },
      updateProject (project) {
        this.$emit('updateProject', project)
      },
      deleteComment (id) {
        this.$emit('deleteProject', id)
      }
    }
  }
</script>

<style>
  .side-bar {
    background-color: rgba(34, 34, 34, 0.8);
  }
  .add-button {
    font-family: inherit; /* override system font */
    grid-area: add-button;
    width:100%;
    color: #fff;
    font-size:18pt;
    background-color:#3a7ce2;
    outline:none;
    border:none;
    cursor:pointer;
  }
  .glow-button:hover {
    animation: none;
    border-color: #3a7ce2;
    color: #3a7ce2;
  }
  .glow-button:active {
    color: #6af;
  }
  .glow-button.disabled {
    border: 2px dashed #6384b7;
    color: #6384b7;
    cursor: default;
  }
  @keyframes glow-button {
    from { color: #326fce; border-color: #326fce }
    to { color: #3a7ce2; border-color: #3a7ce2 }
  }
  .project-form {
    position: absolute;
    padding: 10px;
    width:340px;
    background-color: #3a7ce2;
    transform: translate(-20%, -40%) scale(0);
    will-change:transform;
    transition: transform 150ms ease-in-out;
    box-shadow: 0 0 5px #444;
  }
  .project-form.open {
    transform: scale(1) translate(0);
    transform: translate(0, 0) scale(1);
    z-index: 2;
  }
  textarea {
    margin-top:10px;
    height:200px;
  }
  .project {
    -webkit-user-select: none;
    user-select: none;
    box-sizing:border-box;
    background-color:rgba(255, 255, 255, 0.1);
    margin-top:1px;
    padding:0 0 0 8px;
    max-height:42px;
    transition: max-height 0.4s linear;
    will-change: max-height;
    position:relative;
    overflow:auto;
  }
  .project .project-author {
    font-size:10pt;
    font-weight:lighter;
    color:#ddd;
    margin-top:8px;
    display:none;
    opacity:0;
    transition: opacity 0.4s linear;
  }
  .project .project-title-editor {
    display:block;
  }
  .project .project-title-editor,
  .project .project-title {
    margin-top:12px;
  }
  .project .project-title {
    width: 220px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .project .project-author-avatar {
    float:right;
    width:40px;
    height:40px;
    padding:0;
  }
  .project .project-description-editor,
  .project .project-description {
    font-size:11pt;
  }
  .project .project-description {
    display:none;
    padding-top:4px;
    clear:both;
    opacity:0;
    transition: opacity 0.4s linear;
  }
  .project .project-description ul {
    padding-left:16px;
  }
  .project .action-button-bar {
    width:100%;
    background-color: rgba(0, 0, 0, 0.2);
  }
  .edit.action-button,
  .save.action-button {
    color: #a2d6ff;
    float:right;
  }
  .delete.action-button {
    color:#f77;
  }
  .project .action-button {
    display:inline-block;
    padding:4px 10px;
  }
  .project .action-button-bar a {
    font-weight:lighter;
    font-size:smaller;
  }
  .project .project-title-bar {
    float:left;
  }
  .project.selected {
    padding:8px;
    max-height:400px;
    background-color:rgba(255, 255, 255, 0.2);
  }
  .project.selected .project-author {
    display:inline-block;
  }
  .project.selected .project-description {
    display:block;
  }
  .project.selected .project-description,
  .project.selected .project-author {
    opacity:1;
  }
  .project.selected .project-title {
    width:170px;
    /* undo the ellipsis configuration, so we don't truncate the title */
    white-space: normal;
    overflow: auto;
    text-overflow: unset;
  }
  .project.selected .project-author-avatar {
    width:76px;
    height:76px;
    padding:2px;
  }
  .details {
    text-align:center;
  }
  /* css loader in left column */
  .loader {
    margin: 50px auto 0;
    width: 70px;
    text-align: center;
  }
  .loader > div {
    width: 18px;
    height: 18px;
    background-color: rgba(255,255,255,.8);

    border-radius: 100%;
    display: inline-block;
    -webkit-animation: zoom 1.4s infinite ease-in-out both;
    animation: zoom 1.4s infinite ease-in-out both;
  }
  .loader .loader1 {
    -webkit-animation-delay: -0.32s;
    animation-delay: -0.32s;
  }
  .loader .loader2 {
    -webkit-animation-delay: -0.16s;
    animation-delay: -0.16s;
  }
  @keyframes zoom {
    0%, 80%, 100% {
      -webkit-transform: scale(0);
      transform: scale(0);
    } 40% {
      -webkit-transform: scale(1.0);
      transform: scale(1.0);
    }
  }
</style>
