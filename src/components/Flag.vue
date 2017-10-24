<template>
    <svg viewBox="0 0 60 60">
        <path :fill="color" d="M51.421,22.593L10,3.362V1c0-0.553-0.448-1-1-1S8,0.447,8,1v3v33v3v19c0,0.553,0.448,1,1,1s1-0.447,1-1V40.681  l41.366-16.251c0.373-0.146,0.622-0.501,0.634-0.9C52.012,23.13,51.784,22.762,51.421,22.593z M10,38.533V37V5.566l38.46,17.857  L10,38.533z" />
    </svg>
</template>

<script>
export default {
    props: {
        color: {
            type: String,
            default: '#D80027',
            validator (val) {
                return /^#[0-9a-zA-Z]{3,6}$/.test(val);
            }
        }
    }
};
</script>
