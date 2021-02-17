<template>
  <div class="accordion" :class="{'open':open, 'has-child' : hasChild}">

        <div class="accordion-trigger" @click='open=!open'>
            {{title}}
        </div>

        <ul class="accordion-content" v-show='open'>
            <slot></slot>
        </ul>

    </div>
</template>

<script>

export default {
    name:'accordion',
    data(){
        return {
            name:"accordion",
            open: this.isOpen || false
        }
    },
    props:{
        title: String,
        type: String,
        hasChild: Boolean,
        isOpen: Boolean
    }
}
</script>

<style lang="scss" scoped>
.accordion{

    border-bottom: 1px solid rgba(#000000, 0.1);

    &-trigger{
        position: relative;
        cursor: pointer;
        text-transform: uppercase;
        font-family: 'Montserrat', Arial, Helvetica, sans-serif;
        font-size: 14px;
        line-height: 17px;
        padding: 20px 0;
        font-weight: 600;

        &:before, &:after{

            content:"";
            display: block;
            height: 2px;
            width: 10px;
            background: #000000;
            position: absolute;
            right: 0;
            top: 50%;
            transform: translate3d(-50%, 0, 0);
            transition:0.3s ease transform,

        }

        &:after{

            transform: translate3d(-50%, 0, 0) rotate(270deg);

        }

    }

    &-content{

        overflow: hidden;
        max-height: 0;
        height: 100%;
        transition:.3s ease max-height;

    }

    &.has-child{

        .accordion{

            padding-left:25px;

        }

        .accordion.open{

            .accordion-content{

                max-height: 200px;

            }

        }

    }


    &.open{

        > .accordion-trigger{

            &:after{

                transform: translate3d(-50%, 0, 0) rotate(0);

            }

        }

        .accordion-content{

            max-height: 3000px;
            overflow-y: auto;

        }

    }

}
</style>
