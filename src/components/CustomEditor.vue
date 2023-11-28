<template>
  <div class="editor__wrapper">
    <h1>hello custom editor</h1>
    <div class="toolbar">
      <button class="linkButton" @click="addLink">Link</button>
      <button class="italic" @click="toggleItalic">Italic</button>
      <button class="bold" @click="toggleBold">Bold</button>
    </div>

    <!-- <textarea
      class="textarea"
      rows="10"
      placeholder="write your text"
    ></textarea> -->

    <div id="editor" class="editor" contenteditable="true">
      <p>Select some text and use the buttons to add or remove a link.</p>
    </div>
  </div>

  <p>{{ content }}</p>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";

const content = ref("");

function closestAncestor(node, tagName) {
  while (node && node.tagName !== tagName) {
    node = node.parentNode;
  }
  return node;
}

function checkLinkStatus() {
  const selection = window.getSelection();
  const range = selection.getRangeAt(0);

  const linkButton = document.querySelector(".linkButton");
  const boldButton = document.querySelector(".bold");
  const italicButton = document.querySelector(".italic");

  // Check if the selection contains a link
  const existingLink = closestAncestor(range.commonAncestorContainer, "A");
  const existingBold = closestAncestor(range.commonAncestorContainer, "STRONG");
  const existingItalic = closestAncestor(range.commonAncestorContainer, "EM");

  if (existingLink) {
    linkButton.textContent = "Unlink";
  } else {
    linkButton.textContent = "Link";
  }

  if (existingBold) {
    boldButton.textContent = "Unbold";
  } else {
    boldButton.textContent = "Bold";
  }
  if (existingItalic) {
    italicButton.textContent = "Unitalic";
  } else {
    italicButton.textContent = "Italic";
  }
}

function addLink() {
  const selection = window.getSelection();
  const range = selection.getRangeAt(0);

  // Check if the selection contains a link
  const existingLink = closestAncestor(range.commonAncestorContainer, "A");

  if (existingLink) {
    removeLink(existingLink);
    checkLinkStatus();
  } else {
    if (!range.collapsed) {
      const link = document.createElement("a");
      link.href = prompt("Enter link URL:", "http://");

      // const fragment = range.extractContents();

      // Surround the fragment contents with the new link
      // link.appendChild(fragment);

      const fragment = range.cloneContents();
      range.deleteContents();

      // Insert the new link in place of the extracted contents
      range.insertNode(link);

      // Collapse the range to the end of the link
      range.collapse(false);

      // Remove the selection and add the modified range back
      selection.removeAllRanges();
      selection.addRange(range);
    }
  }
}

function removeLink(elemenet) {
  const range = document.createRange();
  range.selectNodeContents(elemenet);
  const fragment = range.extractContents();
  elemenet.parentNode.replaceChild(fragment, elemenet);
}

// ===================
function toggleBold() {
  const selection = window.getSelection();
  const range = selection.getRangeAt(0);
  const existingElement = closestAncestor(
    range.commonAncestorContainer,
    "STRONG"
  );

  if (existingElement) {
    removeLink(existingElement);
    checkLinkStatus();
  } else {
    const strong = document.createElement("strong");
    strong.appendChild(range.extractContents());
    range.insertNode(strong);
  }
}

function toggleItalic() {
  const selection = window.getSelection();
  const range = selection.getRangeAt(0);
  const existingElement = closestAncestor(range.commonAncestorContainer, "EM");

  if (existingElement) {
    removeLink(existingElement);
    checkLinkStatus();
  } else {
    const em = document.createElement("em");
    em.appendChild(range.extractContents());
    range.insertNode(em);
  }
}

function updateContent() {
  content.value = editor.innerHTML;
}

onMounted(() => {
  document.addEventListener("selectionchange", function () {
    checkLinkStatus();
  });
});
</script>

<style>
.editor__wrapper {
  max-width: 500px;
  width: 100%;
  margin: auto;
}
.toolbar {
  max-width: 500px;
  width: 100%;
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 30px;
}

.textarea {
  width: 100%;
  padding: 5px;
  border: 1px solid #ddd;
  font-size: 16px;
}

.textarea::placeholder {
  font-size: 16px;
}

.textarea:focus {
  outline: none;
}

button {
  padding: 5px 12px;
  background: transparent;
  border: 1px solid #ddd;
  cursor: pointer;
  margin-right: 10px;
}
</style>
