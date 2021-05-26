<template>
    <div class="vs-dark show-file-icons show-folder-icons">
      <div class="workspaceContainer" v-if="rootNode">
        <MonacoTree 
          :directory="rootNode"
          :treeConfig="treeConfig"
          :getActions="getActions"
          @onClickFile="onClickFile"
        />
      </div>
    </div>
</template>

<script>
import { onMounted, unref, ref } from 'vue';
import { MonacoTree, TreeDnD, generateDirectoryTree, FileTemplate, directoryListing, Action, Separator } from "../monaco-tree";

export default {
  name: 'App',

  components: {
    MonacoTree,
  },

  setup() {
    const rootNode = ref(null);
    const treeConfig = ref(null);
    const rootDirectoryName = ref("demo");

    onMounted(() => {
        treeConfig.value = {
            dataSource: {
              /**
               * Returns the unique identifier of the given element.
               * No more than one element may use a given identifier.
               */
              getId: function(tree, element){
                return element.key;
              },
      
              /**
               * Returns a boolean value indicating whether the element has children.
               */
              hasChildren: function(tree, element){
                return element.isDirectory;
              },
      
              /**
               * Returns the element's children as an array in a promise.
               */
              getChildren: function(tree, element){
                return Promise.resolve(element.children);
              },
      
              /**
               * Returns the element's parent in a promise.
               */
              getParent: function(tree, element){
                return Promise.resolve(element.parent);
              },
            },

            renderer: {
              getHeight: function(tree, element){
                return 24;
              },

              renderTemplate: function(tree, templateId, container) {
                return new FileTemplate(container);
              },

              renderElement: function(tree, element, templateId, templateData) {
                templateData.set(element);
              },

              disposeTemplate: function(tree, templateId, templateData) {
                FileTemplate.dispose();
              }
            },

            //tree config requires a controller property but we would defer its initialisation
            //to be done by the MonacoTree component
            //controller: createController(this, this.getActions.bind(this), true),
            dnd: new TreeDnD()
        };

        rootNode.value = generateDirectoryTree(directoryListing, unref(rootDirectoryName))
    });

    const getActions = (file, event) => {
      const actions = [];

      // Directory options
      if (file.isDirectory) {
        actions.push(new Action("1", "New File", "", true, () => {
          console.log("action New File on " + file.name);
        }));

        //menu separator
        actions.push(new Separator());

        actions.push(new Action("2", "New Directory", "", true, () => {
          console.log("action New Directory on " + file.name);
        }));

        actions.push(new Action("3", "Upload Files", "", true, () => {
          console.log("action Upload Files on " + file.name);
        }));
          
      }
      
          
      actions.push(new Action("4", "Download", "", true, () => {
        console.log("action Download on " + file.name);
      }));
      
      actions.push(new Action("5", "Delete", "", true, () => {
        console.log("action Delete on " + file.name);
      }));


      return actions;
    }

    const onClickFile = (file) => {
      if (file.isDirectory) {
        return;
      }

      if (Date.now() - this.lastClickedTime < 500 && this.lastClickedFile === file) {
        this.onDoubleClickFile(file);
      } 
      else {
        console.log(file.name + " clicked");
      }

      this.lastClickedTime = Date.now();
      this.lastClickedFile = file;
    };

    const onDoubleClickFile = (file) => {
      console.log(file.name + " double clicked");
    };

    return {
      rootNode,
      treeConfig,
      rootDirectoryName,
      getActions,
      onClickFile,
      onDoubleClickFile,
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
