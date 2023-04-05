<template>
  <div class="flex justify-end">
    <button
      type="button"
      class="rounded-full bg-indigo-600 p-1.5 text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
      @click="open = true"
    >
      <PlusIcon class="h-5 w-5" aria-hidden="true" />
    </button>
    <TransitionRoot as="template" :show="open">
      <Dialog as="div" class="relative z-10" @close="open = false">
        <div class="fixed inset-0" />

        <div class="fixed inset-0 overflow-hidden">
          <div class="absolute inset-0 overflow-hidden">
            <div
              class="pointer-events-none fixed inset-y-0 right-0 flex max-w-full pl-10"
            >
              <TransitionChild
                as="template"
                enter="transform transition ease-in-out duration-500 sm:duration-700"
                enter-from="translate-x-full"
                enter-to="translate-x-0"
                leave="transform transition ease-in-out duration-500 sm:duration-700"
                leave-from="translate-x-0"
                leave-to="translate-x-full"
              >
                <DialogPanel class="pointer-events-auto w-screen max-w-md">
                  <div
                    class="flex h-full flex-col divide-y divide-gray-200 bg-white shadow-xl"
                  >
                    <div
                      class="flex min-h-0 flex-1 flex-col overflow-y-scroll py-6"
                    >
                      <div class="px-4 sm:px-6">
                        <div class="flex items-start justify-between">
                          <DialogTitle
                            class="text-base font-semibold leading-6 text-gray-900"
                            >Notes</DialogTitle
                          >
                          <div class="ml-3 flex h-7 items-center">
                            <button
                              type="button"
                              class="rounded-md bg-white text-gray-400 hover:text-gray-500 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                              @click="open = false"
                            >
                              <span class="sr-only">Close panel</span>
                              <XMarkIcon class="h-6 w-6" aria-hidden="true" />
                            </button>
                          </div>
                        </div>
                      </div>
                      <div>
                        <div
                          class="block p-4 bg-gray border border-gray-200 rounded-lg shadow bg-gray-100 dark:bg-gray-800 dark:border-gray-700 dark:bg-gray-700 group"
                        >
                        
                          <div v-if="notes && notes.length">
                            <!--Notes list starts here-->
                            <div
                              v-for="(note, index) in notes"
                              :key="index"
                              class="mt-2 block max-w-sm p-4 bg-white border border-gray-200 rounded-lg shadow hover:bg-gray-100 dark:bg-gray-800 dark:border-gray-700 dark:hover:bg-gray-700 group"
                            >
                            {{ alphabet }}
                              <CollectionList
                                :note="note"
                                :index="index"
                                @emitData="emitData"
                              />
                              <div>
                                <!--Edit notes starts here-->
                                <CollectionEdit
                                  v-if="editInput.uid == note.uid"
                                  :note="editInput"
                                  class="sm:mt-[15px]"
                                  @edit="edit"
                                />
                              </div>
                              <!-- Edit notes ends here-->
                            </div>
                          </div>
                          <div v-else>No Notes found</div>
                        </div>
                      </div>
                    </div>
                    <div class="flex flex-shrink-0 justify-end px-4 py-4">
                      <!--Add notes starts here-->
                      <CollectionAdd @add="add" />
                    </div>
                  </div>
                </DialogPanel>
              </TransitionChild>
            </div>
          </div>
        </div>
      </Dialog>
    </TransitionRoot>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import {
  Dialog,
  DialogPanel,
  DialogTitle,
  TransitionChild,
  TransitionRoot,
} from "@headlessui/vue";
import { XMarkIcon, PlusIcon } from "@heroicons/vue/24/outline";

const open = ref(false);
// Notes props
interface NoteProps {
  url: string;
  editUrl: string;
  entity: string; // Entity for the api call
  entityId: string; // Entity id for the api call
  projectId: string; // Project id
}
// Notes props default data
const props = withDefaults(defineProps<NoteProps>(), {
  url: "",
  editUrl: "",
  entity: "",
  entityId: "",
  projectId: "",
});
const notes = ref([]);
// Get notes
const { data: notesdata } = await useAuthLazyFetch(`${props.url}`, {});
notes.value = notesdata.value;
// To edit selected note
const editInput = ref([]);
// Add Notes
const add = async (note: any) => {
  await useAuthLazyFetchPost(
    `${props.editUrl}/${props.entity}/${props.entityId}`,
    {
      body: {
        entity_id: props.entityId,
        project_id: props.projectId,
        note,
        entity: props.entity,
      },
    }
  );
  getNotes();
};
// Edit notes
const edit = async (note: any) => {
  await useAuthLazyFetchPut(`${props.editUrl}/${note.uid}`, {
    body: {
      entity_id: props.entityId,
      project_id: props.projectId,
      note: note.note,
      entity: props.entity,
    },
  });
  getNotes();
};

// Edit and Delete events
const emitData = (note: Object) => {
  note.value == "edit" ? (editInput.value = note.note) : deleteNote(note);
};

// Get notes
const getNotes = async () => {
  const { data: notesdata } = await useAuthLazyFetch(`${props.url}`, {});
  notes.value = notesdata.value;
  editInput.value = [];
};

// Delete notes
const deleteNote = async (note: any) => {
  await useAuthLazyFetchDelete(`${props.editUrl}/${note.note.uid}`, {});
  notes.value.splice(note.index, 1);
  await getNotes();
};
</script>
