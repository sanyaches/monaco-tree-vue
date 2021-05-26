<template>
  <div class="fill" ref="container" />
</template>

<script>
import { createController } from "./monaco-controller";
import { Tree } from "./monaco-utils";
import { onBeforeUnmount, onMounted, watch, ref, unref } from 'vue';

export default {
  name: 'MonacoTree',

  props: {
    directory: {
      type: String,
      default: '',
    },

    treeConfig: {
      type: Array,
      default: () => [],
    }
  },

  setup(props) {
    const container = ref(null);
    const tree = ref(null);

    const ensureTree = () => {
      if (unref(container).lastChild) {
        unref(container).removeChild(unref(container).lastChild);
      }

      const { treeConfig, getActions } = props;

      treeConfig.controller = createController({container: unref(container)}, getActions, true),
  
      tree.value = new Tree(unref(container), treeConfig );
    };
      
    const expandTree = (tree) => {
      const model = tree.model;
      const elements = [];

      let item;
      const nav = model.getNavigator();

      while (item === nav.next()) {
        elements.push(item);
      }

      for (let i = 0, len = elements.length; i < len; i++) {
          model.expand(elements[i]);
      }
    };

      
    const onLayout = () => {
      tree.value.layout();
    };

    onMounted(() => {
        ensureTree();

        tree.value.model.setInput(props.directory);
        tree.value.model.onDidSelect((e) => {
          if (e.selection.length) {
            props.onClickFile(e.selection[0]);
          }
        });

        document.addEventListener("layout", onLayout);
    });

    onBeforeUnmount(() => {
      document.removeEventListener("layout", onLayout);
    }),


    watch(props.directory, (newDirectory, oldDirectory) => {
      if (newDirectory !== oldDirectory) {
        tree.value.model.setInput(newDirectory);
      } else {
        tree.value.model.refresh();
        expandTree(tree);
      }
    });

    return {
      tree,
      container,
    }

  },
}
</script>