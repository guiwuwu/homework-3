<!-- eslint-disable vue/valid-template-root -->
<template>
  <div class="box">
    <div class="title">自动生成小学四则运算题目</div>
    <div class="input">
      <span>请输入要生成的题目数量（大于0）:</span>
      <input type="number" placeholder="" v-model="num" />
    </div>
    <div class="input">
      <span>请输入题目的数值范围（大于0）:</span>
      <input type="number" placeholder="" v-model="range" />
    </div>
    <button class="product" @click="product">生成</button>
    <div class="test">
      <div v-for="(item, index) in math" :key="index">
        题目{{ index + 1 }}: &nbsp;&nbsp;&nbsp;{{ reduce(item[0]) }}
        <span>{{
          item[2] == 0 ? "+" : item[2] == 1 ? "-" : item[2] == 2 ? "*" : "/"
        }}</span>
        {{ reduce(item[1]) }}
      </div>
    </div>
    <div class="ans">
      请输入答案:<br />
      <div v-for="(item, index) in ans" :key="index">
        <input v-model="item[1]" type="text" :placeholder="index + 1" />
      </div>
    </div>
    <button class="submit" @click="submit">提交答案</button>
  </div>
</template>

<script setup>
import fs from "file-saver";
import { ref } from "vue";
// 答案文件的内容
let answersArr = ref([]);
// 题目文件的内容
let fileContent = ref([]);
// 答对个数文件的内容
let gradeConecnt = ref([]);
// 数量
const num = ref();
// 范围
const range = ref();
// 表达式
let math = ref([]);
// 答案
let ans = ref([]);
// 小数化为分数
const reduce = (num) => {
  let int_part = Math.trunc(num); //整数
  let float_part = num - Math.trunc(num); //小数
  if (float_part == 0) return num;
  let arr = [Math.ceil(float_part * 100), 100];
  let arr1 = [arr[0], arr[1]];
  while (arr1[0] != 1) {
    if (arr1[1] % arr1[0] == 0) {
      arr = [arr[0] / arr1[0], arr[1] / arr1[0]];
      arr1 = [arr[0], arr[1]];
    } else arr1 = [arr1[1] % arr1[0], arr1[0]];
  }
  return int_part == 0
    ? `${arr[0]}/${arr[1]}`
    : `${int_part}'${arr[0]}/${arr[1]}`;
};

// 生成数字和字符
const product = () => {
  // console.log(eval("1+1"));
  console.log((16 / 7).toFixed(5) === (3.2 / 1.4).toFixed(5));
  if (num.value > 0 || range.value > 0) {
    math.value = [];
    ans.value = [];
    fileContent.value = [];

    for (let i = 0; i < num.value; i++) {
      //生成两个随机数和运算符 保留两位小数
      let fu = Math.trunc(Math.random() * 3);
      let num1 = (Math.random() * range.value).toFixed(2);
      let num2 = (Math.random() * range.value).toFixed(2);
      fileContent.value.push(
        fu == 0
          ? `${i + 1}.四则运算题目${i + 1}  ${reduce(num1)}+${reduce(num2)}\n`
          : fu == 1
          ? `${i + 1}.四则运算题目${i + 1}  ${reduce(num1)}-${reduce(num2)}\n`
          : fu == 2
          ? `${i + 1}.四则运算题目${i + 1}  ${reduce(num1)}*${reduce(num2)}\n`
          : `${i + 1}.四则运算题目${i + 1}  ${reduce(num1)}/${reduce(num2)}\n`
      );

      math.value.push([num1, num2, fu]);
      // 存入答案
      if (fu == 0)
        ans.value.push([
          Number((Number(num1) + Number(num2)).toFixed(2)),
          null,
        ]);
      else if (fu == 1)
        ans.value.push([Number((num1 - num2).toFixed(2)), null]);
      else if (fu == 2)
        ans.value.push([Number((num1 * num2).toFixed(2)), null]);
      else {
        // ans:[正确答案，我的答案]
        ans.value.push([
          Number((num1 * 10000) / (num2 * 10000) / 10000).toFixed(2),
          null,
        ]);
      }
      answersArr.value.push(`${i + 1}.答案${i + 1}  ${ans.value[i][0]}\n`);
    }
    // 写入文件
    var file = new File(fileContent.value, "Exercises.txt", {
      type: "text/plain;charset=utf-8",
    });
    fs.saveAs(file);
    var file2 = new File(answersArr.value, "Answers.txt", {
      type: "text/plain;charset=utf-8",
    });
    fs.saveAs(file2);
  } else {
    alert("请输入合法的数字!");
    return;
  }
};
const submit = () => {
  let rightArr = [],
    wrongArr = [];
  let right = 0,
    wrong = 0;
  for (let i = 0; i < ans.value.length; i++) {
    if (ans.value[i][1] == null || ans.value[i][1].trim() == "") {
      alert("请做完题目!");
      return;
    } else if (ans.value[i][0] == Number(eval(ans.value[i][1]).toFixed(2))) {
      right++;
      rightArr.push(i + 1);
    } else {
      wrong++;
      wrongArr.push(i + 1);
    }
  }
  gradeConecnt.value = [
    `Correct: ${right}(${rightArr})\n`,
    `Wrong: ${wrong}(${wrongArr})\n`,
  ];
  var file = new File(gradeConecnt.value, "Grade.txt", {
    type: "text/plain;charset=utf-8",
  });
  fs.saveAs(file);
};
</script>

<style>
.box {
  width: 70%;
  margin: 0 auto;
}
.box .title {
  width: 300px;
  margin: 0 auto;
}
.input {
  width: 300px;
  margin-top: 20px;
}
.product {
  margin-top: 20px;
}
.test {
  width: 300px;
  margin-top: 20px;
}
.ans {
  margin-top: 20px;
}
.submit {
  margin-top: 20px;
}
</style>
