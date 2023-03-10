<template>
    <div
        v-if="isVisible(visibility)"
        :style="[
            {
                visibility: isHidden(visibility) ? 'hidden' : undefined
            },
            unref(style) ?? {}
        ]"
        :class="{ feature: true, milestone: true, done: unref(earned), ...unref(classes) }"
    >
        <component :is="unref(comp)" />
        <Node :id="id" />
    </div>
</template>

<script lang="tsx">
import "components/common/features.css";
import Node from "components/Node.vue";
import type { StyleValue } from "features/feature";
import { isHidden, isVisible, jsx, Visibility } from "features/feature";
import type { GenericMilestone } from "features/milestones/milestone";
import { coerceComponent, isCoercableComponent, processedPropType, unwrapRef } from "util/vue";
import type { Component, UnwrapRef } from "vue";
import { defineComponent, shallowRef, toRefs, unref, watchEffect } from "vue";

export default defineComponent({
    props: {
        visibility: {
            type: processedPropType<Visibility | boolean>(Number, Boolean),
            required: true
        },
        display: {
            type: processedPropType<UnwrapRef<GenericMilestone["display"]>>(
                String,
                Object,
                Function
            ),
            required: true
        },
        style: processedPropType<StyleValue>(String, Object, Array),
        classes: processedPropType<Record<string, boolean>>(Object),
        earned: {
            type: processedPropType<boolean>(Boolean),
            required: true
        },
        id: {
            type: String,
            required: true
        }
    },
    components: {
        Node
    },
    setup(props) {
        const { display } = toRefs(props);

        const comp = shallowRef<Component | string>("");

        watchEffect(() => {
            const currDisplay = unwrapRef(display);
            if (currDisplay == null) {
                comp.value = "";
                return;
            }
            if (isCoercableComponent(currDisplay)) {
                comp.value = coerceComponent(currDisplay);
                return;
            }
            const Requirement = coerceComponent(currDisplay.requirement, "h3");
            const EffectDisplay = coerceComponent(currDisplay.effectDisplay || "", "b");
            const OptionsDisplay = coerceComponent(currDisplay.optionsDisplay || "", "span");
            comp.value = coerceComponent(
                jsx(() => (
                    <span>
                        <Requirement />
                        {currDisplay.effectDisplay != null ? (
                            <div>
                                <EffectDisplay />
                            </div>
                        ) : null}
                        {currDisplay.optionsDisplay != null ? (
                            <div class="equal-spaced">
                                <OptionsDisplay />
                            </div>
                        ) : null}
                    </span>
                ))
            );
        });

        return {
            comp,
            unref,
            Visibility,
            isVisible,
            isHidden
        };
    }
});
</script>

<style scoped>
.milestone {
    width: calc(100% - 10px);
    min-width: 120px;
    padding-left: 5px;
    padding-right: 5px;
    background-color: var(--locked);
    border-width: 4px;
    border-radius: 5px;
    color: rgba(0, 0, 0, 0.5);
}

.milestone.done {
    background-color: var(--bought);
    cursor: default;
}

.milestone :deep(.equal-spaced) {
    display: flex;
    justify-content: center;
}

.milestone :deep(.equal-spaced > *) {
    margin: auto;
}
</style>
