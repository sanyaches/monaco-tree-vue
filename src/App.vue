<template>
  <div class="vs-dark show-file-icons show-folder-icons wrapper">
    <div class="workspaceContainer" v-if="rootNode">
      <MonacoTree 
        :directory="rootNode"
        :tree-config="treeConfig"
        :get-actions="getActions"
        @on-click-file="onClickFile"
      />
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { MonacoTree, TreeDnD, generateDirectoryTree, FileTemplate, directoryListing, Action, Separator } from "./components/monaco-tree";

export default {
  name: 'App',

  components: {
    MonacoTree,
  },

  setup() {
    const rootNode = ref(null);
    const treeConfig = ref(null);
    const rootDirectoryName = ref("demo");
    const lastClickedFile = ref(null);
    const lastClickedTime = ref(null);

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
          getHeight: function(){
            return 24;
          },

          renderTemplate: function(tree, templateId, container) {
            return new FileTemplate(container);
          },

          renderElement: function(tree, element, templateId, templateData) {
            templateData.set(element);
          },

          disposeTemplate: function() {
            FileTemplate.dispose();
          }
        },

        //tree config requires a controller property but we would defer its initialisation
        //to be done by the MonacoTree component
        //controller: createController(this, this.getActions.bind(this), true),
        dnd: new TreeDnD()
      };

      rootNode.value = generateDirectoryTree(directoryListing, rootDirectoryName.value)
    });

    const getActions = (file) => {
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
    
    const onDoubleClickFile = (file) => {
      console.log(file.name + " double clicked");
    };

    const onClickFile = (file) => {
      if (file.isDirectory) {
        return;
      }

      if (Date.now() - lastClickedTime.value < 500 && lastClickedFile.value === file) {
        onDoubleClickFile(file);
      } 
      else {
        console.log(file.name + " clicked");
      }

      lastClickedTime.value = Date.now();
      lastClickedFile.value = file;
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
.wrapper {
  width: 300px;
  height: 600px;
  position: relative;
  margin:0 auto;
}
</style>
